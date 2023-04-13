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

아래는 legacy backup server 에서 호출하는 crontab 코드
```php
# cat crontab -l

0 9 * * * cd /root/backup_firstmall_log && wget https://domain.com/classname1/method
```

상기 코드는 /root/backup_firstmall_log 폴더로 이동 후 firstmall web server 의 html 파일을 가져오는 명령어이다. 이것만으로 [firstmall web server] 호출 할 수 있으며, 호출된 URL 에 해당하는 컨트롤러는 실행되므로 충분했다. 아래는 [firstmall web server] 의 백업 코드이다.

```php
class classname1 extends front_base
{
    private $hostname = null;
    private $userId = null;
    private $password = null;
    private $database = null;
    private $dumpPathPrefix = ROOTPATH . "data/data/database_backup/";
    private $dumpPath = null;
    private $schemaPath = ROOTPATH . "custom/database/";

    public function method() {
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

crontab 에 작성해 둔 명령어대로 실행해보면 /root/back_firstmall_log 폴더에 method, method.1 파일이 생성되며, 내용은 아래와 같이 성공여부, 테이블명, 용량으로 백업 로그를 남긴다.

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