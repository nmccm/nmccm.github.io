---
title: "Goutte Client Example"
categories: 
- PHP
- Laravel
tags:
- Laravel
- Goutte Client
---

간단한 PHP 웹 스크레이퍼인 Goutte 라이브러리를 사용하여 특정 사이트에 로그인하고, 로그인 세션을 얻어야만 접근 가능한 화면을 스크래핑 해본 과정을 여기에 기록한다. Goutte 공식(https://github.com/FriendsOfPHP/Goutte)사이트에서는 아래와 같이 설명한다.

> Goutte is a screen scraping and web crawling library for PHP. Goutte provides a nice API to crawl websites and extract data from the HTML/XML responses. WARNING: This library is deprecated. As of v4, Goutte became a simple proxy to the HttpBrowser class from the Symfony BrowserKit component. To migrate, replace Goutte\Client by Symfony\Component\BrowserKit\HttpBrowser in your code.

```php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Goutte\Client;
use Symfony\Component\HttpClient\HttpClient;

class GoutteClientExampleController extends Controller
{
    public function index(Request $request) {
        try {

            $userAgent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36';
            $client = new Client(HttpClient::create(array(
                'headers' => array(
                    'user-agent' => $userAgent,
                    'Accept' => 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
                    'Accept-Language' => 'en-US,en;q=0.5',
                    'Referer' => preg_replace('login', '', $this->getUrl()),
                    'Upgrade-Insecure-Requests' => '1',
                    'Save-Data' => 'on',
                    'Pragma' => 'no-cache',
                    'Cache-Control' => 'no-cache',
                ),
            )));
            $client->setServerParameter('HTTP_USER_AGENT', $userAgent);

            $req = $client->request($this->getDefaultMethod(), $this->getUrl());
            $form = $req->selectButton((String)$this->getSubmitButtonTxt())->form();
            $form[$this->getNameTx()] = $this->getName();
            $form[$this->getPassTx()] = $this->getPass();

            $crawler2 = $client->submit($form);
            // echo $crawler2->filterXPath('html/head/title')->text()."\n";
            // $orgBody = $crawler2->filterXPath('html/body')->text();
            // log::debug(print_r($orgBody, true));
            $descriptions = $crawler2->filter('table.contentTable')->each(function($node) {
                // return $node->html();
                log::debug(print_r($node->html(), true));
            });
            // echo '<pre>';
            // $r = get_class_methods($crawler);
            // print_r($form);
            // echo '</pre>';

            return view('temp.index', [
                'client' => $client,                
            ]);
        }
        catch(\Exception $e) {
            log::error(__METHOD__ . " : " . $e->getMessage());
        }
    }

    public function getName() {
        return 'login id'; // login id
    }

    public function getPass() {        
        return $this->getName() . '!@#';    // login password
    }

    public function getNameTx() {
        return 'username';  // user id form name
    }

    public function getPassTx() {
        return 'password'; // user password form name
    }

    public function getSubmitButtonTxt() {
        return '로그인'; // submit button text
    }

    public function getDefaultMethod() {
        return 'GET';
    }

    public function getUrl() {
        return 'http://domain/login'; // url
    }
}

```

