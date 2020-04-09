### PHP URL 재작성(rewrite)
최근 PHP 웹사이트는 URL에 PHP 파일명을 노출하지 않는다.   
  
### URL 재작성 설정 - phpinfo() -> mod_rewrite 확인
```
sudo a2enmod rewrites
```
### .htaccess 파일 root 경로에 작성, 수정
/var/www/html$ vi .htaccess
```
RewriteRule ^.*$ /index.php [NC,L,QSA]
```
### conf 파일에 내용 추가 - 그리고 작업할 root 경로 변경 가능
```
$ sudo vi /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
	~
    ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/php_teacher/17_framework3/public

        <Directory /var/www/html>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Require all granted
        </Directory>
	~	
sudo systemctl restart apache2  
```
# Validation
email 중복 체크를 위한 쿼리 - DatabaseTable.php  
email 중복 체크, 빈칸 체크  
password - one way hashing function  
password_hash() - 원하는 문자열을 암호화
password_verify() - 복호화는 간단합니다. ( 암호화된 문자를 푼다는 의미 )
대신 복호화된 문자를 볼 순 없고 비교만 가능합니다. 비교하여 true 또는 false 를 반환합니다.

> password_verify('비교할 문자', $hash);

이런식으로 하면 됩니다.
PASSWORD_DEFAULT 기본 알고리즘  
> 데이터베이스에 저장하기 전에 비밀번호를 해시화  
> $author['password'] = password_hash($author['password'], PASSWORD_DEFAULT);  
  
> 21_cookieSession 410   
# Cookie
- PHP 스크립트에서 PHP스크립트 URL에 접속하면 setcookie() 내장 함수 호출  
- 쿠키명과 값을 HTTP set-cookie 헤더에 담아 브라우저에 전송. ex) 쿠키명 mycookie, 값 value  
- 브라우저는 HTTP 헤더를 읽고 mycookie 쿠키에 value를 저장한다  
- 이후 페이지를 요청할 때마다 브라우저는 mycookie=value를 HTTP쿠키 헤더에 추가한다  
- 페이지 요청에 HTTP 쿠키 헤더가 있으면 PHP는 자동으로 $_COOKIE 배열에 쿠키 정보를 할당한다  
- $_COOKIE['mycookie'] 에 'value' 문자열이 저장된다  
```
setcookie ( string $name [, string $value = "" [, int $expires = 0 [, string $path = "" [, string $domain =  " [, bool $secure = FALSE [, bool $httponly = FALSE ]]]]]] )
```
- **path** '/admin/'  지정하면 admin과 하위 디렉터리 페이지를 요청할 때만 쿠키 정보를 전달한다. 마지막 / 는 디렉터리명을 정확히 지정하는 역활을 하며, 생략하면 /adminfake/ 등 /admin으로 시작하는 모든 경로에서 쿠키에 접근할 수 있다.
- **domain** 인수도 비슷하다. 지정한 도메인 외에는 쿠키에 접근하지 못하도록 차단한다.
- www.example.com, support.example.com 등 여러 도메인에 사용하려면 '.example.com'을 지정하여 쿠키를 공유하도록 허용한다. 
- **secure** 인수를 1로 지정하면 SSL(secure socket layer) 접속, 즉 https:// 로 시작하는 URL을 요청할 때만 쿠키를 전송
- **httpOnly** 인수를 1로 지정하면 브라우저와 자바스크립트가 쿠키에 접근할 수 없다. 일반적으로 자바스크립트는 현재 페이지에 설정된 쿠키를 읽고 다양한 방식으로 활용한다. 혹여 쿠키 데이터에 민감한 개인정보가 담겨 있다면 막대한 피해를 입는다. httpOnly를 1로 설정하면 PHP 스크립트는 평소와 똑같이 쿠키를 브라우저로 전송하지만 브라우저의 자바스크립트는 쿠키를 볼 수 없다.
- **name**을 제외한 모든 인수는 생략할 수 있다. 그러나 인수에 값을 지정하려면ㄴ 그 앞에 선언된 모든 인수에 값을 지정해야 한다. 예로 domain 인수를 지정하려면 expiryTime 인수도 값을 지정해야 한다. 이때 문자열 인수 (value, path, domain)는 빈 문자열을, 숫자 인수(expiryTime, secure)는 0을 전달하면 인수 생략과 같은 효과를 낸다.f

- 쿠키 삭제하기
1)setcookie('Name');
2)setcookie('Name', time()-3600);

ex)
```
if(!isset($_COOKIE['visits'])){
    $_COOKIE['visits'] = 0;
}
$visits = $_COOKIE['visits'] + 1;
setcookie('visits', $visits, time() + 3600 * 24 * 365);
// time() 유닉스 타임스탬프(unix timestamp) 1970년 1월1일 부터 32비트 정수
// 3600초(60초 * 60분), 24시간 , 365일.
// ex) 쿠키 만료일 20년 이후로 설정. setcookie('mycookie', 'somevalue', time() + 3600 * 24 * 365 *20);
// but,,  32비트 정수 최댓값을 날짜로 환산하면 2038년 1월 19일이다. 이후 시각은 오류가 발생한다. (참고)

if($visits > 1){
    echo "$visits 번째 방문하셨습니다. (COOKIE)<br>";
}else{ // 첫 방문
    echo "웹사이트에 오신 걸 환영합니다. 둘러보려면 여기를 클릭하세요! (COOKIE)<br>";
}
```


# session
> php.ini 확인  
```
session.save_handler = files;
session.save_path = "/tmp"; 
session.use_cookies = 1;
```
session.save_path 는 세션 추적용 임시 파일이 저장될 경로다. 현재 디렉토리는 "/var/lib/php/sessions";  

> 세션 시작  
session_start();  
> 세션 변수에 값을 할당  
$_SESSION['password'] = 'mypassword';  
> 세션 변수를 제거 **unset()** 함수  
unset($_SESSION['password']);  
> 세션을 마치고 모든 세션 변수를 삭제 **session_destroy()** 함수   
$_SESSION = [];  
session_destroy();  

ex)
```
session_start();
if (!isset($_SESSION['visits'])) {
    $_SESSION['visits'] = 0;
}
$_SESSION['visits'] = $_SESSION['visits'] + 1;
// 세션은 보존 기간을 계산할 필요가 없는 대신 브라우저를 닫는 즉시 모든 데이터가 사라진다.

if ($_SESSION['visits'] > 1) {
    echo $_SESSION['visits'] . "번째 방문하셨습니다. (SESSION)<br>";
} else { // 첫 방문
    echo "(SESSION) 웹사이트에 오신 걸 환영합니다. 둘러보려면 여기를 클릭하세요!";
}
```
isLoggedIn() 함수는 로그인 사용자가 접근할 때만 참을 반환
Authentication 클래스로 로그인 여부를 검사
EntryPoint.php 사용자 로그인 상태를 판단

### Mysql management
> 24_mysql 455   

**mysqlpump** 데이터베이스 백업 명령  
```
mysqlpump -u ijdbuser -pmypassword ijdb > 20200320Ijdb.sql
```
-u 로그인ID : MySQL 서버 사용자명  
-pmypassword : -p와 비밀번호는 붙여쓴다. 그렇지 않으면 실행 후 비밀번호를 별도로 입력해야 한다.  
ijdb : 백업할 스키마  
명령어 뒤에 > 연산자를 붙이면 명령 실행 결과가 파일로 저장된다.   
경로 없이 파일명만 지정하여 실행하면 실행한 디렉터리에 백업 파일이 생성된다.  
/var/backups/ijdb.sql 처럼 전체경로를 지정해되 된다 .

**mysql** 데이터베이스 복원 명령 
```
mysql -u ijdbuser -pmypassword ijdb < 20200320Ijdb.sql
```
> 바이너리 로그 증분 백업   
- Select문을 제외한 모든 INSERT, UPDATE, DELETE문이 기록된다.  
- MySQL 워크벤치나 mysqlpump로 만든 백업 파일을 먼저 복원하고, 백업 시점 이후에 변경된 데이터를 바이너리 로그로 복구한다.  
- my.cnf 파일에 서버 설정이 저장된다. 설정 파일이 없으면 기본설정이 적용되므로 새로 파일을 만들고 바이너리 로그 설정을 추가해야 한다. 명령줄 편집기로 고쳐야 한다  
- /etc/mysql/mysql.conf.d/mysqld.cnf
```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
1. log_bin
>> #log_bin                 = /var/log/mysql/mysql-bin.log
=> # 주석을 제거한다.
>> log_bin                 = /var/log/mysql/mysql-bin.log
2. server-id 
>> #server-id               = 1
=> # 주석을 제거한다.
>> server-id               = 1
3. MySQL 재시작
```
sudo systemctl restart mysql.service
```
4. log_bin 파일 경로 : ls /var/log/mysql
5. mysqlbinlog : MySQL 바이너리 로그 데이터를 일반적인 SQL 명령어로 변환한다.
```
mysqlbinlog binlog.000041 binlog.000042 > binlog.sql
mysql -u root -pabde1245 < binlog.sql
```

**MySQL 비밀번호 분실**
```
sudo systemctl stop mysql.service

sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
>> **my.cnf** 파일에서 [ mysqld ]를 찾고 skip-grant-tables 설정을 추가한다    
>> MySQL 서버의 모든 계정에 무제한적인 권한을 부여하는 설정.   
>> ***수정후 바로 삭제!!***
```
[ mysqld ]
skip-grant-tables
```
>> 서버를 재시작한다  
```
sudo systemctl start mysql.service
```
>> MySQl 접속 후 비밀번호 재설정한다  
```
UPDATE mysql.user SET Password=PASSWORD('pass1234') WHERE User = 'user-name'
```
>> 서버 정지 후 my.cnf 파일 설정을 다시 원래대로 제거  
```
sudo systemctl stop mysql.service

sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

[ mysqld ]  
# skip-grant-tables  
```
>> 서버를 재시작한다    
```
sudo systemctl start mysql.service
```

**MySQL Index**
>> EXPAIN : SELECT 쿠리를 수행하는 내부 과정을 직접 볼 수 있다.  
```
EXPLAIN SELECT joketext FROM joke WHERE id = 6;
```
>> index  
```
ALTER TABLE `ijdb`.`joke` ADD INDEX `index2` (`authorid` ASC);
```

>> Composite index 다중 인덱스  
```
ALTER TABLE `ijdb`.`jokecategory` ADD INDEX `composite` (`jokeid` ASC, `categoryid` ASC);
```
다음 쿼리는 3,4번 글 카테고리에 포함되는지 검사하는 인덱스를 조회한다  
select * from jokecategory where jokeid =3 and categoryid = 4;  
두 컬럼 중 인덱스 정의에 먼저 나온 컬럼은 단독 인덱스 기능도 수행한다.  
select * from joke_category where jokeid = 1  

**Foreing Key**
외래 키 제약은 테이블을 생성할 때 CREATE TABLE 명령어에 명시한다.  
```
CREATE TABLE joke (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    joketext TEXT,
    jokedate DATE NOT NULL,
    authorID INT,
    FOREIGN KEY (authorID) REFERENCES author (id)
) DEFAULT CHARACTER SET utf8 ENGINE=InnoDB
```
기본테이블은 ALTER TABLE 명령으로 외래 키 제약을 추가할 수 있다  
```
ALTER TABLE joke ADD FOREIGN KEY (authorId) REFERENCES author (id)
```

# 참조변수
- 참조변수는 일반변수와 다르며 윈도우의 바로가기나 리눅스의 심링크symlink와 비슷하다.
- 참조변수를 생성하려면 앰퍼샌드문자&를 붙인다
ex)
```
$originalVariable = 1;
$reference = &$originalVariable;
$originalVariable = 2;
echo $reference;
```
# 투명 캐싱 (transparent caching)
- 캐싱은 데이터를 저장했다가 나중에 빠르게 접근하는 기능으로, 겉으로 봤을 때 내부적으로 캐싱 과정을 알 필요가 없어 투명 캐싱이라 한다

# 카테고리
- 카테고리 테이블 쿼리 
```
CREATE TABLE `ijdb`.`category` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(255) NULL,
    PRIMARY KEY (`id`));
)
```

# ... 연산자 (언패킹 unpacking) or (스플랫 splat) 연산자
$joke = new \Ijdb\Entity\Joke($authorsTable);
Joke의 전체 클래스명은 \Ijdb\Entity\Joke 이며 $this->className 변수에 저장된다. 
이 코드를 이렇게 바꿀수 있다. 두 코드는 똑같이 동작한다.   
$joke = new $this->className($authorsTable);  
그러나 엔티티 클래스는 생성자 인수의 종류와 갯수가 서로 다르다. 예를 들어 Author 엔티티 클래스는 $jokesTable 인스턴트를
배열에 담아 생성자로 전달받고, $this->constructorArgs변수에 담는다.  
**...** 연산자는 언패킹 unpacking 연산자 또는 스플랫 splat 연산자라 하며 여러 인수를 배열로 묶어 전달한다.  
ex)  $array[1,2]; 
someFunction(...$array);   
이코드는   
someFunction(1, 2); 와 똑같다.
참고URL: https://blog.programster.org/php-splat-operator




