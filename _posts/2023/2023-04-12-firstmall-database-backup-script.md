---
title: "Firstmall Database Backup Script"
categories:
- CodeIgniter
tags:
- CodeIgniter
- PHP
- MySQL
- Firstmall
- Linux
- ShellScript
---

코드이그나이터(이하 CI) 프레임워크로 개발된 가비아 쇼핑몰을 유지보수 할 일이 생겨 소스를 보던중, 데이터베이스 백업이 별도로 이루어 지지 않는다는 점을 발견했다. 
쇼핑몰 웹 서버에서 1차 데이리 백업을 받고, 별도 백업 서버에 복제하는 방식으로 이중 백업을 하기로 결정했다. 
쇼핑몰 웹 서버의 데이터 유지기간은 1일, 백업 서버의 데이터는 30일간 보관하기로 했다. (사실 정말 문제가 터지면 큰 도움은 안된다)

웹서버에 가용용량은 한정되어 있으므로 몇일동안의 백업데이터를 보관하기 어려웠다. 하루 데이터양이 1G 가 넘어가므로 30일만 해도 30G 이다. 
1일치만 보관한다는건 그날 당일 문제건에 대해선 대처 가능할지 몰라도 혹 몇일 방치된 버그를 발견한다면 몇일전 데이터를 확인해봐야 될 필요가 있다. 
이런 부분을 감안한다면 최소 30일 정도의 데이터는 보관되어야 한다고 판단되었다.

이러한 일들은 리눅스의 스케쥴러(crontab) 사용하는것이 일반적이지만, 이곳에선 스케쥴러를 사용할 수 없는 상황이었고, 고민 끝에 보안엔 조금 취약하지만 약속된 토큰을 전달 함으로써 웹 호출 방식으로 구현해야만 했다.

웹 호출은 당연히 리눅스 스케쥴러(crontab) 사용이 가능한 백업 서버에서 호출하도록 하였고, 아래는 그 코드이며, backup_firstmall_log 폴더로 이동 후에 wget 으로 웹 호출을 하는 명령이다. 
wget 명령은 웹서버에서 파싱과 브라우저 랜더링까지 마친 내용을 명령어를 실행한 서버에 html 파일로 저장한다.

```php
# cat crontab -l

0 9 * * * cd /root/backup_firstmall_log && wget https://domain.com/path1/backupScript
```

스케쥴러가 실행된후 해당 폴더를 조회하면 아래와 같이 파일이 생성되는 것을 확인할 수 있다.

```php
# ls -al
total 2136
drwxr-xr-x  2 root root  4096 2023-04-13 09:00 .
dr-xr-x--- 14 root root  4096 2023-03-29 10:07 ..
-rw-r--r--  1 root root 19835 2023-03-18 09:00 backupScript
-rw-r--r--  1 root root 19835 2023-03-19 09:00 backupScript.1
-rw-r--r--  1 root root 19838 2023-03-28 09:00 backupScript.10
-rw-r--r--  1 root root 19883 2023-03-29 09:00 backupScript.11
-rw-r--r--  1 root root 19883 2023-03-30 09:00 backupScript.12
-rw-r--r--  1 root root 19883 2023-03-31 09:00 backupScript.13
-rw-r--r--  1 root root 19934 2023-04-01 09:00 backupScript.14
-rw-r--r--  1 root root 19934 2023-04-02 09:00 backupScript.15
-rw-r--r--  1 root root 19936 2023-04-03 09:00 backupScript.16
-rw-r--r--  1 root root 19936 2023-04-04 09:00 backupScript.17
```

데이터베이스 전체의 백업으로 진행하면 파일이 비대해져 이동, 복사, 압축 등 모든것에 시간 비용이 많이 발생하므로 테이블 단위로 백업을 진행하였다.  

```php
# tail -n 10 /roo/backup_firstmall_log/method.1

[create success] table_name1 536766
[create success] table_name1 1085235
[create success] table_name1 2001905
[create success] table_name1 441873
[create success] table_name1 43335598
[create success] table_name1 3322563
[create success] table_name1 2173
```

backupScript 파일 내용은 위와 같이 성공 실패 여부, 테이블명, 용량 형태로 각 테이블별로 백업되도록 구현하였다.
백업을 실제로 실행하는 PHP 코드는 아래와 같다. 

```php
class path1 extends front_base
{
    private $hostname = null;
    private $userId = null;
    private $password = null;
    private $database = null;
    private $dumpPathPrefix = ROOTPATH . "data/data/database_backup/";
    private $dumpPath = null;
    private $schemaPath = ROOTPATH . "custom/database/";

    public function backupScript() {
        try {
            $targetPathAndFile = 'config/database';
            $connectionInfo = file_get_contents(APPPATH . $targetPathAndFile . EXT);
            $connectionInfo = preg_replace('/</', '', $connectionInfo);
            $connectionInfo = preg_replace('/=>/', '', $connectionInfo);
            preg_match_all('/\'(username|password|database|hostname)\'(.+),/i', $connectionInfo, $output);

            if(!isset($output[0]) || empty($output[0]) || !is_array($output[0])) {
                throw new Exception('parse failed ' . $targetPathAndFile);
            }

            foreach($output[0] as $item) {
                $tmp = explode(' ', preg_replace('/,/', '', preg_replace('/\'/', '', $item)));

                if(!is_array($tmp)) {
                    throw new Exception('explode failed : ' . $item);
                }

                substr_count('hostname', $tmp[0]) > 0 && $this->hostname = $tmp[2];
                substr_count('password', $tmp[0]) > 0 && $this->password = $tmp[2];
                substr_count('database', $tmp[0]) > 0 && $this->database = $tmp[2];
                substr_count('username', $tmp[0]) > 0 && $this->userId = $tmp[2];
            }

            if(!$this->isDatabaseConfig()) {
                throw new Exception('config failed');
            }

            $this->dumpPath = $this->dumpPathPrefix . date('Ymd');
            $tables = $this->db->list_tables();

            if(!is_dir($this->dumpPathPrefix)) {
                @mkdir($this->dumpPathPrefix);
                @chmod($this->dumpPathPrefix, 0777);
            }

            if(!is_dir($this->dumpPath)) {
                @mkdir($this->dumpPath);
                @chmod($this->dumpPath, 0777);
            }

            foreach($tables as $n => $table) {
                if($table != 'backup_table') {
                    $cmd = "/usr/local/mysql/bin/mysqldump -u {$this->userId} -p{$this->password} {$this->database} {$table} > {$this->dumpPath}/{$table}.sql 2> /dev/null";
                    $file = $this->dumpPath . '/' . $table . '.sql';
                    exec($cmd, $execOutput, $execResult);
                    echo (is_file($file)) ? '[create success] ' . $table . ' ' . filesize($file) . PHP_EOL : '[fail] ' . $table . PHP_EOL;
                }
            }
        }
        catch(Exception $e) {
            print "Exception : " . $e->getMessage();
            exit;
        }
    }

    private function isDatabaseConfig() {
        return !is_null($this->hostname) && strlen($this->hostname) > 1
            && !is_null($this->password) && strlen($this->password) > 1
            && !is_null($this->database) && strlen($this->database) > 1
            && !is_null($this->userId) && strlen($this->userId) > 1;
    }
}
```

CI 프레임워크에서 사용되는 데이터베이스 설정 파일을 읽어들여 mysqldump 명령어로 백업을 진행하여, 파일의 존재 여부를 체크하여 성공/실패 구분 처리를 하였다.
실 토큰 검증 로직은 보안상 제거 하였다.