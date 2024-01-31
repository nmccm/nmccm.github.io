---
title: "Goutte Client Example"
categories: 
- PHP
tags:
- Laravel
- Goutte Client
---

Goutte Client Example

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
//            $orgBody = $crawler2->filterXPath('html/body')->text();
            // log::debug(print_r($orgBody, true));
            $descriptions = $crawler2->filter('table.contentTable')->each(function($node) {
                // return $node->html();
                log::debug(print_r($node->html(), true));
            });
//            echo '<pre>';
//            $r = get_class_methods($crawler);
//            print_r($form);
//            echo '</pre>';

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

