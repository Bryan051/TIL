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
videoViewCache.put(cacheKey, newVideoView);
```
- 캐싱 이후 db에 직접 조회를 하지 않아 눈에띄게 속도가 빨라졌지만 데이터가 쌓이면 점점 느려지는 것은 변함이 없다.
- 쌓인 데이터를 소모시켜준다. (videoview 는 가장 최근에 시청기록때문에 받아오기때문)
