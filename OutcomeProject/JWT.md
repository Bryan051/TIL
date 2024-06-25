## JWT 생성 및 검증
```
@Component
public class JwtProvider {

    @Value("${secret-key}")
    private String secretkey;

    public String create(String userEmail) {
        Date expiredDate = Date.from(Instant.now().plus(1, ChronoUnit.HOURS));
        Key key = Keys.hmacShaKeyFor(secretkey.getBytes(StandardCharsets.UTF_8));

        return Jwts.builder()
                .signWith(key, SignatureAlgorithm.ES256)
                .setSubject(userEmail)
                .setIssuedAt(new Date())
                .setExpiration(expiredDate)
                .compact();
    }

    public String validate(String jwt) {

        String subject = null;
        Key key = Keys.hmacShaKeyFor(secretkey.getBytes(StandardCharsets.UTF_8));

        try {
            subject = Jwts.parserBuilder()
                    .setSigningKey(key)
                    .build()
                    .parseClaimsJws(jwt)
                    .getBody()
                    .getSubject();

        } catch (Exception exception) {
            exception.printStackTrace();
            return null;
        }
        return subject;

    }
```
- secret-key 는 노출 위험을 막기 위해 application properties에 원문을 두고 가져다 썼다.
- io.jsonwebtoken.security 의 Keys.hmacShaKeyFor 로 secret-key를 bytes[] 로 저장 </br>
  .signWith ES256으로 변환하여 빌드했다. validate 동일.
   
