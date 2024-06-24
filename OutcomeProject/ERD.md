## ERD
<img src="https://github.com/Bryan051/TIL/assets/68111122/0e46e55f-290c-44da-a2d2-f5c2c48f96f6" width="80%" >

```
-- user 테이블 생성
CREATE TABLE user(
    user_id  BIGINT(20)   NOT NULL AUTO_INCREMENT,
    email    VARCHAR(255) NOT NULL,
    username VARCHAR(100) NOT NULL,
    password VARCHAR(255) NOT NULL,
    type     VARCHAR(100) NOT NULL,
    role     VARCHAR(100) NOT NULL DEFAULT 'user',
    PRIMARY KEY (user_id)
);

-- video 테이블 생성
CREATE TABLE video(
    vid_id      BIGINT(20)   NOT NULL AUTO_INCREMENT,
    user_id     BIGINT(20)   NOT NULL,
    vid_name    VARCHAR(255) NOT NULL,
    vid_content TEXT,
    upload_date DATE         NOT NULL,
    ad_time     DATE         NOT NULL,
    PRIMARY KEY (vid_id),
    FOREIGN KEY (user_id) REFERENCES user (user_id)
);

-- ad 테이블 생성
CREATE TABLE ad(
    ad_id   BIGINT(20)   NOT NULL AUTO_INCREMENT,
    ad_type VARCHAR(100) NOT NULL,
    ad_name VARCHAR(255) NOT NULL,
    PRIMARY KEY (ad_id)
);

-- adlist 테이블 생성
CREATE TABLE adlist(
    adlist_id  BIGINT(20) NOT NULL AUTO_INCREMENT,
    vid_id     BIGINT(20) NOT NULL,
    ad_id      BIGINT(20) NOT NULL,
    play_count INT,
    PRIMARY KEY (adlist_id),
    FOREIGN KEY (vid_id) REFERENCES video (vid_id),
    FOREIGN KEY (ad_id) REFERENCES ad (ad_id)
);

-- video_view 테이블 생성
CREATE TABLE video_view(
    vidview_id  BIGINT(20) NOT NULL AUTO_INCREMENT,
    user_id2    BIGINT(20) NOT NULL,
    vid_id      BIGINT(20) NOT NULL,
    last_played DATETIME,
    view_date   DATE,
    PRIMARY KEY (vidview_id),
    FOREIGN KEY (user_id2) REFERENCES user (user_id),
    FOREIGN KEY (vid_id) REFERENCES video (vid_id)
);
```
