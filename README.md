
## PHP Start

### 1. Descrição

Microframework interno para criação de projetos em PHP: [phpstartApiServer](https://github.com/incluirtecnologia/phpstartApiServer)

Considerações sobre [MVC Model 2 e ADR](https://github.com/pmjones/adr).

### 2. Estrutura

```
1── .docker: contém arquivos para configuração e manipulação de serviços do docker
│   ├── apache
│   │   ├── vhost.conf: arquivo de definição do virtual host para apache
│   ├── mysql
│   │   ├── mysql-data: possui arquivos do bd criados pelo docker para o mysql
│   │   ├── .gitignore: ignora a pasta mysql-data
│   ├── nginx
│   │   ├── nginx.conf: equivalente do vhost.conf para nginx
│   ├── php
│   │   ├── php-ini-overrides.ini: arquivo para definição de parâmetros do php.
Ex.: max_file_size post_max_size, max_file_upload, ...

2── src: Código Psr4
│   ├── Controller
│   ├── Middleware
│   ├── Model
│   ├── Service
│   ├── Validator
│   ├── Worker

3── log: contém arquivos de log da aplicação para auxilio no desenvolvimento

4── public:
│   ├── index.php: Front Controller.
│   ├── .htaccess: Configuração de redirecionamento do apache

5── config
│   ├── routes.php: api-routes.php e template-routes.php
│   ├── settings.php: configurações da aplicação
│   ├── settings.local.php: configurações locais da aplicação
│   ├── dependencies.php: definição de dependências

6── data/database/paches: usado para salvar patches do banco de dados.
│   ├── .gitkeep: usado para versionar o diretório vazio (normalmente o git ignora diretórios vazios)

7── tests: ainda não existe, mas deve ser reservada para execução de testes de unidade, funcional e aceitação (via phpunit ou codeception).

├── README.md: arquivo de descrição básica sobre o phpstart.
└── .gitignore
└── composer.json
└── .dockerignore: ignora arquivos e pastas irrelevantes para build de produção (Ex. .git, node_modules, data/database/patches)
└── .editorconfig: tentativa de padronizar a estética do código em diferentes IDE's
└── docker-compose.yml: arquivo de definição de serviços do docker para o desenvolvimento de aplicações utilizando o phpstart.
```

**Observação 1:** As pastas referentes ao _frontend_ deixaram de existir (assets, node_modules, src/Views etc). A pasta app foi removida e seu conteúdo movido para a raiz.

**Observação 2:** O arquivo docker-compose.yml foi simplificado

```yml
version: '3.1'

services:
    mysql:
        image: mysql:5.7.22
        environment:
            - MYSQL_DATABASE=
            - MYSQL_USER=
            - MYSQL_PASSWORD=
            - MYSQL_ROOT_PASSWORD=
        volumes:
            - .:/srv/vhosts/phpApp
            - ./.docker/mysql/mysql-data:/var/lib/mysql
        working_dir: /srv/vhosts/phpApp
        ports:
            - 13306:3306

    beanstalk:
        image: schickling/beanstalkd
        ports:
            - 21300:11300

    php-apache:
        image: marciodojr/phpstart-apache-docker-image:dev
        environment:
            - DEV_MODE=1
            - DB_HOST=
            - DB_PORT=
            - DB_NAME=
            - DB_USER=
            - DB_PASS=
            - APP_SECRET=holly_molly!
        working_dir: /srv/vhosts/phpApp
        volumes:
            - .:/srv/vhosts/phpApp
            - ./.docker/php/php-ini-overrides.ini:/etc/php/7.2/apache2/conf.d/99-overrides.ini
            - ./.docker/apache/vhost.conf:/etc/apache2/sites-available/000-default.conf
        ports:
            - 8888:80
        depends_on:
            - beanstalk
            - mysql
```

### 2. PHPFIG


> We're a group of established PHP projects whose goal is to talk about commonalities between our projects and find ways we can work better together.

PSR's (desde 2011):

| PSR | Descrição                                                 |    PhpStart                    | Slim      |
| --- | -------------------                                       | -----------------------------  | ----      |
| 1   | [Basic Coding Style](https://www.php-fig.org/psr/psr-1/)  | ✓ auto-indent IDE              | ✓ =       |
| 2   | [Coding Style Guide](https://www.php-fig.org/psr/psr-4/)  | ✓ auto-indent IDE              | ✓ =       |
| 4   | [Autoloading](https://www.php-fig.org/psr/psr-4/)         | ✓ composer autoloader          | ✓ =       |
| 3   | [Log](https://www.php-fig.org/psr/psr-3/)                 | 𐄂 quase não utilizado          | ✓ monolog |
| 6   | [Caching](https://www.php-fig.org/psr/psr-6/)             | 𐄂 não utilizado                | 𐄂 =       |
| 7   | [Message](https://www.php-fig.org/psr/psr-7/)             | 𐄂 simplerouter incompatível    | ✓         |
| 11  | [Container](https://www.php-fig.org/psr/psr-11/)          | ✓ pimple                       | ✓ pimple  |
| 13  | [Links](https://www.php-fig.org/psr/psr-13/)              | 𐄂 não utilizado                | 𐄂 =       |
| 15  | [Handlers](https://www.php-fig.org/psr/psr-15/)           | 𐄂 middlewares não conformantes | 𐄂 3 ✓ 4   |
| 16  | [Simple Cache](https://www.php-fig.org/psr/psr-16/)       | 𐄂 não utilizado                | 𐄂         |
|     |                                                           |                                |           |

### 3. Exemplo de rotina


1. Setup
```sh
# em docker-compose.yml
# definir nomes de variaveis de ambiente dos containers
# no terminal
docker-compose up
# em outra aba do terminal
docker exec -it <php-container> sh
composer install # ou composer update
# em outra aba do terminal
docker exec -it <mysql-container> sh
mysql -u <user> -p <database> sh
# colar todos os patches necessários para ter o banco atualizado
```

2. Registro da Rota
```php
// config/routes.php
return [
    [
        'pattern' => '/minha-rota/([0-9a-z_]+)',
        'middlewares' => [
            /*
            callable, ou
            MiddlewareClass::class . ':nonStaticPublicMethod'
            */
        ],
        'callback' => \IntecPhp\Controller\MyController::class . ':myAction',
        /*
            callable ou
            ControllerClass::class . ':nonStaticPublicMethod'
        */
    ],
    /* outras rotas */
];
```
3. Criação de um Controller

```php
<?php
// src/Controller/MyController.php

namespace IntecPhp\Controller;

use IntecPhp\Model\ResponseHandler;
/**
use Some\Other\Dependency1
use Some\Other\Dependency2
use Some\Other\Dependency3
...
**/

class MyController
{
    /*dependências injetadas via pimple */
    public function __construct($d1, $d2, $d3)
    {
        /*atribuição de dependências*/
    }

    /* $request injetada pelo router */
    public function myAction($request)
    {
        // se GET
        // $params = $request->getQueryParams()
        // se POST
        // $params = $request->getPostParams()
        /* se possui parâmetros de url
         (ex.: /investimentos/1/financiamentos/10) */
        // $params = $request->getPostParams()
        // $params[0] == 1, $params[1] == 10
        // não possui suporte a outros métodos Ex: PUT

        try {
            // valida $params
            // injeta $params nos métodos das dependencias
            // se algo deu errado lança exceção
            // se deu tudo certo preenche ResponseHandler com código 200, 201, 204
            // mensagem e array de dados de resposta
            $rp = new ResponseHandler(200, 'ok', $dataArray);
        } catch(Exception $e) {
            $rp = new ResponseHandler(400, $e->getMessage);
        }

         /* faz um echo json_encode com um array no formato
         [
            'code' => int,
            'message' => string,
            'data' => array
         ]
         */
        $rp->printJson();
    }
}
```

4. Registro das dependências do Controller
```php
// config/dependencies.php

use IntecPhp\Controller\MyController;

$dependencies[MyController::class] = function($c) {
    // injeta as dependencias de MyController e retorna uma instância
    $d1 = $c[Dependency1::class]
    $d2 = $c[Dependency2::class]
    $d3 = $c[Dependency3::class]
    return new MyController($d1, $d2, $d3);
};
```

5. Repetir criação da classe e registro no container para cada próxima camada.

### 4. Comparativos


0. Informações Gerais

![Phpstart Stack](/imgs/phpstart-stack.png)
![Slim Stack](/imgs/slim-stack.png)
![Slim Middlewares](/imgs/middleware.png)



**PhpStart** = **Slim** (ver [slimskeleton](https://github.com/slimphp/Slim-Skeleton))

1. public/index.php

**PhpStart**


```php
<?php

//Everything is relative to the application root now.
chdir(dirname(__DIR__));

$settings = require 'config/settings.php';

if(file_exists('config/settings.local.php')) {
    $settings = array_replace_recursive($settings, require 'config/settings.local.php');

}
if($settings['display_errors']) {
    error_reporting(E_ALL);
    ini_set("display_errors", 1);
};

if (!file_exists('./vendor/autoload.php')) {
    echo 'Please run `composer install` first!';
}

include './vendor/autoload.php';

use Intec\Router\SimpleRouter;
use IntecPhp\Middleware\HttpMiddleware;
use Pimple\Psr11\Container;
use Pimple\Container as PimpleContainer;

SimpleRouter::setRoutes(require 'config/routes.php');
SimpleRouter::setNotFoundFallback(HttpMiddleware::class . ':pageNotFound');
SimpleRouter::setErrorFallback(HttpMiddleware::class . ':fatalError');

$dependencies = new PimpleContainer();
$dependencies['settings'] = $settings;

require 'config/dependencies.php';

SimpleRouter::match($_SERVER['REQUEST_URI'], new Container($dependencies));
```

**Slim**

```php
<?php

//Everything is relative to the application root now.
chdir(dirname(__DIR__));

require './vendor/autoload.php';

$settings = require 'config/settings.php';

if(file_exists('config/settings.local.php')) {
    $settings = array_replace_recursive($settings, require './config/settings.local.php');
}

$app = new \Slim\App([
    'settings' => $settings
]);

require './config/dependencies.php';
require './config/routes.php';

$app->run();
```

2. Routes:

**PhpStart**
```php
// config/routes.php

return [
    [
        'pattern' => '/user/login',
        'callback' => Controller\LoginController::class . ':login',
    ],
    [
        'pattern' => '/virtual-users',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualUserController::class . ':listAll',
    ],
    [
        'pattern' => '/virtual-users/add',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualUserController::class . ':create',
    ],
    [
        'pattern' => '/virtual-users/remove',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualUserController::class . ':delete',
    ],
    [
        'pattern' => '/virtual-domains',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\DomainController::class . ':listAll',
    ],
    [
        'pattern' => '/virtual-domains/add',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\DomainController::class . ':create',
    ],
    [
        'pattern' => '/virtual-domains/edit',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\DomainController::class . ':update',
    ],
    [
        'pattern' => '/virtual-domains/remove',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\DomainController::class . ':delete',
    ],
    [
        'pattern' => '/virtual-aliases',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualAliasController::class . ':listAll',
    ],
    [
        'pattern' => '/virtual-aliases/add',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualAliasController::class . ':create',
    ],
    [
        'pattern' => '/virtual-aliases/remove',
        'middlewares' => [
            Middleware\Auth::class,
        ],
        'callback' => Controller\VirtualAliasController::class . ':delete',
    ]
];
```

**Slim**

```php
// config/routes.php

$app->post('/user/login', LoginController::class . ':login');

$app->group('', function(){
    // crud domínios
    $this->group('/virtual-domains', function () {
        $this->get('', DomainController::class . ':listAll');
        $this->post('', DomainController::class . ':create');
        $this->patch('/{id:[0-9]+}', DomainController::class . ':update');
        $this->delete('', DomainController::class . ':delete');
    });

    // crud emails
    $this->group('/virtual-users', function() {
        $this->get('', VirtualUserController::class . ':listAll');
        $this->post('', VirtualUserController::class . ':create');
        $this->delete('', VirtualUserController::class . ':delete');
    });

    // crud aliases
    $this->group('/virtual-aliases', function(){
        $this->get('', VirtualAliasController::class . ':listAll');
        $this->post('', VirtualAliasController::class . ':create');
        $this->delete('', VirtualAliasController::class . ':delete');
    });
})->add(Auth::class . ':process');

```

3. Middlewares

**PhpStart**
```php
<?php

namespace IntecPhp\Middleware;

use IntecPhp\Model\Account;
use IntecPhp\Model\ResponseHandler;

class AuthenticationMiddleware
{
    private $account;

    public function __construct(Account $account)
    {
        $this->account = $account;
    }

    public function isAuthenticated($request)
    {
        if (!$this->account->isLoggedIn()) {
            $rp = new ResponseHandler(403, 'Você não tem permissão para acessar este recurso');
            $rp->printJson();
            exit;
        }
    }
}
```

**Slim**

```php
<?php

namespace Mdojr\EmailProvider\Middleware;

use Mdojr\EmailProvider\Service\Account;

class Auth
{
    use \Mdojr\EmailProvider\Helper\JsonResponse;

    private $account;

    public function __construct(Account $account)
    {
        $this->account = $account;
    }

    public function process($request, $response, $next)
    {
        $header = $request->getHeader('Authorization');
        $token = $header ? $this->getToken($header[0]) : '';
        $data = $this->account->get($token);
        if (!$token || !$data) {
            return $this->toJson($response, 403, 'Você não possui permissão para acessar este recurso');
        }
        $req = $request->withAttribute('auth', (array)$data);
        return $next($req, $response);
    }

    private function getToken($header)
    {
        if(preg_match("/Bearer\s(\S+)/", $header, $matches)) {
            return $matches[1];
        }
        return null;
    }
}
```

4. Controllers

**PhpStart**
```php
<?php

namespace Mdojr\EmailProvider\Controller;

use Mdojr\EmailProvider\Service\Database\VirtualDomain;
use Mdojr\EmailProvider\Model\ResponseHandler;
use Exception;

class DomainController
{
    private $vdomain;

    public function __construct(VirtualDomain $vdomain)
    {
        $this->vdomain = $vdomain;
    }

    // GET /virtual-domains
    public function listAll()
    {
        try {
            $vdomainData = $this->vdomain->fetchAll();
            $rp = new ResponseHandler(200, 'ok', $vdomainData);
        } catch(Exception $ex) {
            $rp = new ResponseHandler(400, $ex->getMessage());
        }
        $rp->printJson();
    }

    // POST /virtual-domains/add
    public function create($request)
    {
        $params = $request->getPostParams();
        try {
            $domain = $this->vdomain->create($params['name']);
            $rp = new ResponseHandler(200, 'ok', $domain);
        } catch(Exception $ex) {
            $rp = new ResponseHandler(400, $ex->getMessage());
        }
        $rp->printJson();
    }

    // POST /virtual-domains/edit
    public function update($request)
    {
        $params = $request->getPostParams();
        try {
            $domain = $this->vdomain->update($params['id'], $params['name']);
            $rp = new ResponseHandler(200, 'ok', $domain);
        } catch(Exception $ex) {
            $rp = new ResponseHandler(400, $ex->getMessage());
        }
        $rp->printJson();
    }

    // POST /virtual-domains/remove
    public function delete($request)
    {
        $params = $request->getPostParams();
        try {
            $this->vdomain->delete($params['id']);
            $rp = new ResponseHandler(200);
        } catch(Exception $ex) {
            $rp = new ResponseHandler(400, $ex->getMessage());
        }
        $rp->printJson();
    }
}
```

**Slim**

```php
<?php

namespace Mdojr\EmailProvider\Controller;

use Mdojr\EmailProvider\Service\Database\VirtualDomain;
use Exception;

class DomainController
{
    use \Mdojr\EmailProvider\Helper\JsonResponse;

    private $vdomain;


    public function __construct(VirtualDomain $vdomain)
    {
        $this->vdomain = $vdomain;
    }

    // GET /virtual-domains
    public function listAll($request, $response)
    {
        try {
            $vdomainData = $this->vdomain->fetchAll();
            return $this->toJson($response, 200, 'ok', $vdomainData);
        } catch(Exception $ex) {
            return $this->toJson($response, 400, $ex->getMessage());
        }
    }

    // POST /virtual-domains
    public function create($request, $response)
    {
        $params = $request->getParams();
        try {
            if(empty($params['name'])) {
                throw new Exception('Domínio não informado');
            }
            $domain = $this->vdomain->create($params['name']);
            return $this->toJson($response, 200, 'ok', $domain);
        } catch(Exception $ex) {
            return $this->toJson($response, 400, $ex->getMessage());
        }
    }

    // PATCH /virtual-domains
    public function update($request, $response, $args)
    {
        $params = $request->getParams();
        try {
            if(empty($params['name'])) {
                throw new Exception('Domínio não informado');
            }
            $domain = $this->vdomain->update($args['id'], $params['name']);
            return $this->toJson($response, 200, 'ok', $domain);
        } catch(Exception $ex) {
            return $this->toJson($response, 400, $ex->getMessage());
        }
    }

    // DELETE /virtual-domains
    public function delete($request, $response)
    {
        $params = $request->getParams();
        try {
            if(empty($params['domains'])) {
                throw new Exception('Nenhum domínio foi informado');
            }
            $this->vdomain->delete($params['domains']);
            return $this->toJson($response, 204);
        } catch(Exception $ex) {
            return $this->toJson($response, 400, $ex->getMessage());
        }
    }
}
```

5. Frontend

As respostas do servidor de api são idênticas tanto para o PhpStart quanto para o Slim e seguem o formato conhecido:

```json
{
    code: <http-status-code>,
    message: <reason>,
    data: <array>
}
```

6. Testes Funcionais (Slim)

```php
    public function testListDomainWithoutToken()
    {
        $response = $this->runApp('GET', '/virtual-domains');
        $this->check403Response($response);
    }
```

```php
    protected function runApp($requestMethod, $requestUri, $requestData = null, $token = null)
    {
        $environment = Environment::mock(
            [
                'REQUEST_METHOD' => $requestMethod,
                'REQUEST_URI' => $requestUri
            ]
        );
        $request = Request::createFromEnvironment($environment);
        if (isset($requestData)) {
            $request = $request->withParsedBody($requestData);
        }
        if($token) {
            $request = $request->withHeader('Authorization', sprintf('Bearer %s', $token));
        }

        return $this->app->process($request, new Response());
    }
```
```php
    protected function check403Response(Response $response)
    {
        $body = $this->decodeResponse($response);
        $this->assertSame(403, $response->getStatusCode());
        $this->assertSame(403, $body['code']);
        $this->assertSame('Você não possui permissão para acessar este recurso', $body['message']);
        $this->assertSame([], $body['data']);
    }
```

Links Importantes:

Documentação do Slim Framework: https://www.slimframework.com/docs/

## Frontend

Microframework interno para construção de frontends em html, css e js.

[PhpStartWebApp](https://github.com/incluirtecnologia/phpstartwebapp)


0. Estrutura

```
1── .docker: contém arquivos para configuração e manipulação de serviços do docker
│   ├── apache
│   │   ├── vhost.conf: arquivo de definição do virtual host para apache
│   ├── nginx
│   │   ├── nginx.conf: equivalente do vhost.conf para nginx
│   ├── php
│   │   ├── php-ini-overrides.ini: arquivo para definição de parâmetros do php.
Ex.: max_file_size post_max_size, max_file_upload, ...

2── app
│   ├── config:
│   │   ├── json: arquivos json para simulação de retorno do backend
│   │   json-mapper.php: array de mapeamento de rotas para json's
│   │   template-routes.php: rotas de páginas da aplicação

3── src: Código Psr4
│   ├── {Middleware, Service, View}

4── views:
│   ├── partial: contém arquivos html parciais (layout, sidebar, ...)
│   ├── template contém o conteúdo central de cada página

5── assets:
│   ├── fonts: contém fontes específicas que não podem ser adicionadas via npm
│   ├── {img,js,sass}

5── public:
│   ├── {fonts,img,js,css}
├── {README.md,.babelrc,dockerignore,.editorconfig,.gitignore}
├── {composer.json,docker-compose.yml,gruntfile.js,package.json}
```

**Observação 1:** As pastas referentes ao _backend_ deixaram de existir (.docker/mysql, app/src/Controller, ...).

**Observação 2:** O arquivo docker-compose.yml foi simplificado.

```yml
version: '3.1'

services:
    node:
        image: node:10.6.0-alpine
        container_name: phpstartwa-node
        environment:
            - SERVERNAME=webserver
        volumes:
            - .:/srv/vhosts/phpApp
        working_dir: /srv/vhosts/phpApp
        command: /bin/sh -c "npm install grunt-cli && npm install && npm run dev"
        ports:
            - 3000:3000
            - 3001:3001
        depends_on:
            - webserver

    php-fpm:
        image: marciodojr/phpstart-apache-docker-image:dev
        container_name: phpstartwa
        environment:
            - APP_SECRET=holly_molly!
        working_dir: /srv/vhosts/phpApp
        volumes:
            - .:/srv/vhosts/phpApp
            - ./.docker/php/php-ini-overrides.ini:/etc/php/7.2/apache2/conf.d/99-overrides.ini
            - ./.docker/apache/vhost.conf:/etc/apache2/sites-available/000-default.conf
        ports:
            - 2999:80
```