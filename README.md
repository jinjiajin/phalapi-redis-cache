

## 1 使用方法

1. 使用composer下载 jinjiajin/cache 组件：
    ```
    composer install jinjiajin/cache

    ```

2. 在MVC框架`Model`层或者`Service`层的基类添加`__call()`方法，例如：
    ```
    #如：src/app/Model/Base.php
    <?php
    namespace App\Model;
    use jinjiajin\cache\Cache;

    class Base extends \PhalApi\Model\DataModel {

    /**
     * 切换数据库
     * @return \PhalApi\Database\NotORMDatabase
     */
    protected function getNotORM() {
        return \PhalApi\DI()->notorm;
    }
    public function __call($name, $arguments)
    {
        $redis = \PhalApi\DI()->cache;
        // ***主要是这一行***
        return (new Cache($redis))->get($this, $name, $arguments);
    }
    }
    ```
    其中，`jinjiajin\cache\Cache`对象创建时必须传入 Redis 连接句柄，
    然后再通过`get`方法获取缓存。

3. 让子类继承`Base`类，例如：
    ```
    namespace App\Model;

    class StoreCategory extends Base
    {
        public function getById($id)
        {
            return ['id' => 100, 'title' => 'PHP documents']
        }
    }

    ```
4. 然后就可以在`Controllers`类或其他`Model`类中使用：
    ```
    (new StoreCategory)->getById(100);           // 原始的、不用缓存的调用方式，一般是读取数据库的数据。
    (new StoreCategory)->getByIdCache(100);      // 使用缓存的调用方式，缓存键名为：App_Model_StoreCategory:getById: + md5(参数列表)
    (new StoreCategory)->getByIdClear(100);      // 删除这个缓存
    (new StoreCategory)->getByIdFlush();         // 删除 getById() 方法对应的所有缓存，即删除 App_Model_StoreCategory:getById:*。这个方法不需要参数。
    ```

## 2 构造函数参数

`jinjiajin\cache\Cache`构造函数参数：
- `$redis`: 必须。Redis连接对象。
- `$config`: 可选。缓存的一些配置。
    - `$config['prefix']`: 缓存键名前缀，默认空字符串。
    - `$config['expire']`：缓存过期时间，默认`3600`秒。
    - `$config['emptyExpire']`：原函数返回空值是的缓存过期时间，默认`10`秒。

## 3. 方法
除了构造函数，主要就是2个方法。

### 3.1 `get()`方法
获取缓存，如果未缓存，则调用原对象的原方法，拿到数据后保存到Redis中，并返回数据。
有3个参数，基本就是固定的：
- `$object`: 当前调用的类对象，传入`$this`。
- `$name`： 欲调用的方法名称，由`__call`自动获取后传入。
- `$arguments`: 欲调用的方法的参数，由`__call`自动获取后传入。

### 3.2 `expire()`方法
设置缓存过期时间。
默认的缓存过期时间是`3600`秒，也可以用`expire()`方法动态设置。
只有1个参数：
- `$time`: 缓存过期时间，单位为秒。