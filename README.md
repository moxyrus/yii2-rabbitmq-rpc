Yii2 RabbitMQ RPC component
===========================

The component is designed to quickly and easily start using the RabbitMQ queue server in the yii2 application. 
Parallel execution of tasks is the main goal of its creation.

INSTALLATION
------------

```bash
composer require mamatveev/yii2-rabbitmq-rpc:@dev
```

Add a component configuration in the application config:

```php
return [
    'components' => [
        'rpc' => [
            'class' => 'mamatveev\yii2rabbitmq\RabbitComponent',
            'host' => '127.0.0.1',
            'port' => 5672,
            'user' => 'guest',
            'password' => 'guest'
        ],
];
```

USAGE
-----

To test the capabilities of a component, create a simple console controller

```php
<?php

namespace app\commands;

use mamatveev\yii2rabbitmq\RabbitComponent;
use yii\console\Controller;

class RpcController extends Controller
{

       public function actionRabbitServer()
    {
        /** @var RabbitComponent $rpc */
        $rpc = \Yii::$app->rpc;

        $rpcServer = $rpc->initServer('gis-search');

        $callback = function ($obj) {
            sleep($obj->sleep);
            return $obj->bar();
        };

        $rpcServer->setCallback($callback);
        $rpcServer->start();
    }

    public function actionRabbitClient()
    {

        $ema = new Ema;
        $ema->sleep = 3;

        $ema2 = new Ema;
        $ema2->sleep = 2;

        /** @var RabbitComponent $rpc */
        $rpc = \Yii::$app->rpc;

        // init a client
        $rpcClient = $rpc->initClient('gis-search');
        $rpcClient->addRequest($ema);
        $rpcClient->addRequest($ema2);

        // use callback for responses
        $msg = $rpcClient->getReplies();

        print_r($msg);
    }
}
```

To run RPC server execute bash command:

```bash
./yii rpc/rabbit-server
```

And then in other bash tab run client controller action
 
```bash
./yii rpc/rabbit-client
```
