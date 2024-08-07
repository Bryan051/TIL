## 1만개 데이터 넣는 순간부터 db를 밀어넣는 속도가 급격히 늘어남

```java
@Component
@RequiredArgsConstructor
public class DataLoader implements CommandLineRunner {

    private final VideoWriteRepository videoWriteRepository;
    private final UserWriteRepository userWriteRepository;
    private final AdWriteRepository adWriteRepository;
    private final VideoViewWriteRepository videoViewWriteRepository;
    private final AdViewWriteRepository adViewWriteRepository;
    private final VideoAdWriteRepository videoAdWriteRepository;

    @PersistenceContext(unitName = "writeEntityManagerFactory")
    private EntityManager writeEntityManager;

    @Override
    @org.springframework.transaction.annotation.Transactional
    public void run(String... args) throws Exception {
        Random random = new Random();




                // 더미 데이터 수동으로 더하기
        // vid_id ( user 5명, video 각 5명) 25 중 랜덤 선택, userid(5명) 랜덤 선택,
        long videoCount = videoWriteRepository.count();
        long userCount = userWriteRepository.count();

        // 데이터가 없으면 예외 발생
        if (videoCount == 0 || userCount == 0) {
            throw new IllegalStateException("No videos or users available.");
        }

        // 비디오와 사용자 목록을 미리 로드
        List<Video> videos = videoWriteRepository.findAll();
        List<User> users = userWriteRepository.findAll();

        int totalViews = 3000;
        int viewCount = 0;
        int batchSize = 1000;

        while (viewCount < totalViews) {
            createVideoViews(random, videos, users, batchSize);
            viewCount += batchSize;
            System.out.println("viewCount = " + viewCount);
        }
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createVideoViews(Random random, List<Video> videos, List<User> users, int batchSize) {

        for (int i = 0; i < batchSize; i++) {

            Video video = videos.get(random.nextInt(videos.size()));
            User user = users.get(random.nextInt(users.size()));

            VideoView latestVideoView = videoViewWriteRepository.findTopByUserIdAndVidIdOrderByIdDesc(user, video);

            // 배제
//            if (videoViews.isEmpty()) {
//                VideoView newVideoView = new VideoView();
//                newVideoView.setUserId(user);
//                newVideoView.setVidId(video);
//                newVideoView.setCreatedAt(LocalDate.now());
//                newVideoView.setLast_played(0);
//                videoViewWriteRepository.save(newVideoView);
//            }

            int vidLength = video.getVidLength();
            int requestLastPlayed = random.nextInt(vidLength - 30) + 30;
            int previousLastPlayed = latestVideoView != null ? latestVideoView.getLast_played() : 0;

            int duration = requestLastPlayed < previousLastPlayed
                    ? video.getVidLength() - previousLastPlayed + requestLastPlayed
                    : requestLastPlayed - previousLastPlayed;

            VideoView newVideoView = new VideoView();
            newVideoView.setUserId(user);
            newVideoView.setVidId(video);
            newVideoView.setCreatedAt(LocalDate.now());
            newVideoView.setDuration(duration);
            newVideoView.setLast_played(requestLastPlayed);
            videoViewWriteRepository.save(newVideoView);

            List<VideoAd> videoAds = videoAdWriteRepository.findVideoAdByVideo(video);
            int adCount = 0;
            int maxAdsPerVideoView = 3;

            for (VideoAd videoAd : videoAds) {
                if (videoAd.getAdPosition() > previousLastPlayed && videoAd.getAdPosition() <= requestLastPlayed) {
                    if (adCount >= maxAdsPerVideoView) {
                        break;
                    }
                    AdView adView = new AdView();
                    adView.setCreatedAt(LocalDate.now());
                    adView.setVideoAd(videoAd);
                    adViewWriteRepository.save(adView);
                    adCount++;
                }
            }
        }
        writeEntityManager.flush();
    }
```

문제가 될 만한 부분을 생각 해 보았을 때
- VideoView를 videoViewWriteRepository.save(newVideoView); 에서 계속 만들어주고
- 그게 수천개씩 쌓이는 도중 다시 .findTopByUserIdAndVidIdOrderByIdDesc(user, video);를 조회해와서 생기는 것.</br>
## 캐싱.
```java
private final Map<String, VideoView> videoViewCache = new HashMap<>();

String cacheKey = user.getUserId() + "_" + video.getVidId();
            VideoView latestVideoView = videoViewCache.get(cacheKey);
---
videoViewWriteRepository.save(newVideoView);
// 캐시에 새로운 VideoView 객체를 추가 (덮어쓰기)
videoViewCache.put(cacheKey, newVideoView);
```
- 캐싱 이후 db에 직접 조회를 하지 않아 눈에띄게 속도가 빨라졌다.
- 캐시에 cacheKey에 해당하는 새로운 VideoView 객체를 추가 (덮어쓰기)하기 때문에 용량도 크게 늘어나지 않는다
    - cacheKey 값이 userId,VideoId 값인데 현재 프로젝트에서 5 * 5 = 25의 조합이 끝.
    - 추후 CACHE_SIZE_LIMIT를 사용하여 캐시의 크기를 제한할 수 있다. Cache.size() > Limit -> remove(oldestKey)
 
## 3만개정도부터 속도가 크게 느려진다
![image](https://github.com/user-attachments/assets/53369f1f-9dc8-4a12-9c75-2b75589d1e5a)

- userid, videoid 조합을 미리 짜 놓고 cacheKey로 저장 후 가져오는 것으로, 중복되는 랜덤 값 지정 및 cacheKey 생성을 막아보려한다
- videoad 조합도 매번 조회보다 메모리에 불러놓고 맞는 값을 찾는게 현명한 방법일 듯 하다.
  ```java
  @PersistenceContext(unitName = "writeEntityManagerFactory")
    private EntityManager writeEntityManager;

    private final Map<String, VideoView> videoViewCache = new HashMap<>();
    private final int CACHE_SIZE_LIMIT = 10000;

    @Override
    @org.springframework.transaction.annotation.Transactional
    public void run(String... args) throws Exception {
        Random random = new Random();

        // 더미 데이터 더하기
        // vid_id ( user 5명, video 각 5명) 25 중 랜덤 선택, userid(5명) 랜덤 선택,
        long videoCount = videoWriteRepository.count();
        long userCount = userWriteRepository.count();

        // 데이터가 없으면 예외 발생
        if (videoCount == 0 || userCount == 0) {
            throw new IllegalStateException("No videos or users available.");
        }

        // 비디오와 사용자 목록을 미리 로드
        List<Video> videos = videoWriteRepository.findAll();
        List<User> users = userWriteRepository.findAll();

        // 모든 사용자 id와 비디오id 조합의 캐시 키를 생성
        List<String> cacheKeys = new ArrayList<>();
        for (User user : users) {
            for (Video video : videos) {
                String cacheKey = user.getUserId() + "_" + video.getVidId();
                cacheKeys.add(cacheKey);
                videoViewCache.put(cacheKey, null); // 초기화
            }
        }

        // 모든 VideoAd 데이터를 미리 로드
        Map<Long, List<VideoAd>> videoAdsCache = new HashMap<>();
        for (Video video : videos) {
            videoAdsCache.put(video.getVidId(), videoAdWriteRepository.findVideoAdByVideo(video));
        }

        int createViews = 100000;
        int batchSize = totalViews / cacheKeys.size(); // 각 조합당 처리할 작업 수

        for (String cacheKey : cacheKeys) {
            for (int i = 0; i < batchSize; i++) {
                createVideoViews(random, cacheKey, videos, users, videoAdsCache);
            }
        }
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createVideoViews(Random random, String cacheKey, List<Video> videos,
                                 List<User> users, Map<Long, List<VideoAd>> videoAdsCache) {
        
        // 해당하는 캐시 키의 조합 한번에 위 batchSize for문이 반복되고 사이즈는 원하는 createView 갯수/조합갯수
        String[] ids = cacheKey.split("_");
        Long userId = Long.parseLong(ids[0]);
        Long videoId = Long.parseLong(ids[1]);

        User user = users.stream().filter(u -> u.getUserId().equals(userId)).findFirst().orElseThrow();
        Video video = videos.stream().filter(v -> v.getVidId().equals(videoId)).findFirst().orElseThrow();

        VideoView latestVideoView = videoViewCache.get(cacheKey);


        int vidLength = video.getVidLength();
        int requestLastPlayed = random.nextInt(vidLength - 30) + 30;
        int previousLastPlayed = latestVideoView != null ? latestVideoView.getLast_played() : 0;

        int duration = requestLastPlayed < previousLastPlayed
                ? video.getVidLength() - previousLastPlayed + requestLastPlayed
                : requestLastPlayed - previousLastPlayed;

        VideoView newVideoView = new VideoView();
        newVideoView.setUserId(user);
        newVideoView.setVidId(video);
        newVideoView.setCreatedAt(LocalDate.now());
        newVideoView.setDuration(duration);
        newVideoView.setLast_played(requestLastPlayed);
        videoViewWriteRepository.save(newVideoView);
        // 캐시에 새로운 VideoView 객체를 추가 (덮어쓰기)
        videoViewCache.put(cacheKey, newVideoView);

        // 로드 된 ad 중 vidid 맞는 값만 가져옴, video마다 광고가 붙어있는 지점이 달라 필요하다.
        List<VideoAd> videoAds = videoAdsCache.get(video.getVidId());
        int adCount = 0;
        int maxAdsPerVideoView = 3;

        for (VideoAd videoAd : videoAds) {
            if (videoAd.getAdPosition() > previousLastPlayed && videoAd.getAdPosition() <= requestLastPlayed) {
                if (adCount >= maxAdsPerVideoView) {
                    break;
                }
                AdView adView = new AdView();
                adView.setCreatedAt(LocalDate.now());
                adView.setVideoAd(videoAd);
                adViewWriteRepository.save(adView);
                adCount++;
            }
        }
        writeEntityManager.flush();
        }
    }
  ```

## 폐기. 단위가 넘어갈수록 db 부하가 감당이 안된다. 정합성에는 맞겠지만 더미데이터 넣는 과정이니..
직접 insert 문으로 넣기로 한다
