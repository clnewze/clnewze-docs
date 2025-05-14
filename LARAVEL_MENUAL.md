# Laravel EC2 배포 메뉴얼 (기존 빌드 파일 기준)
작성일: 2025-05-14

이 메뉴얼은 **이미 빌드된 Laravel 프로젝트**를 Amazon Linux 기반 EC2 서버에 HTTPS 환경으로 배포하기 위한 단계입니다.
Laravel 8.4, PHP 7.4 기준입니다.

---

## 1. Laravel 프로젝트 압축 및 EC2 업로드

### ✅ 로컬에서 프로젝트 압축
```bash
zip -r laravel-app.zip laravel-app
```

### ✅ EC2 서버로 전송
```bash
scp laravel-app.zip ec2-user@<EC2-IP>:/var/www/
```

### ✅ EC2에서 압축 해제
```bash
cd /var/www/
unzip laravel-app.zip
```

---

## 2. 퍼미션 및 Laravel 캐시 설정

### ✅ 디렉터리 권한 설정
```bash
cd /var/www/laravel-app
sudo chown -R nginx:nginx .
chmod -R 775 storage
chmod -R 775 bootstrap/cache
```

### ✅ Artisan 캐시 설정 (선택)
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## 3. Nginx + PHP-FPM 연동 설정

### ✅ Nginx 설정 파일 생성
**/etc/nginx/conf.d/laravel.conf**
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    root /var/www/laravel-app/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

### ✅ Nginx 및 PHP-FPM 재시작
```bash
sudo systemctl restart php-fpm
sudo systemctl restart nginx
```

---

## 4. HTTPS 적용 (Let’s Encrypt 인증서 발급)

### ✅ certbot 설치
```bash
sudo yum install -y certbot python3-certbot-nginx
```

### ✅ SSL 인증서 발급 및 자동 구성
```bash
sudo certbot --nginx -d yourdomain.com
```

---

## 5. 확인 사항

- 도메인 DNS가 EC2 공인 IP로 연결되어 있어야 함
- 보안 그룹에서 포트 80, 443 오픈
- Laravel `.env` 파일 포함 여부 확인
- `vendor/`, `public/` 디렉터리 완전성 확인

---

## ✅ 완료
이제 브라우저에서 https://yourdomain.com 으로 접속하여 Laravel 화면이 출력되는지 확인합니다.
