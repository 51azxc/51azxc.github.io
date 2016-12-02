title: "Laravel ORM部分练习"
date: 2016-01-12 18:15:27
tags: "orm"
categories: ["php", "laravel"]
---

### 获取id

> [Data van from database = ErrorException Undefined property](http://stackoverflow.com/questions/23009678/data-van-from-database-errorexception-undefined-property)

默认的`get()`方法返回的是一个记录的集合，如果需要得到一条记录的id，可以使用如下方法：
```php
$this->select('id')->where('property', '=', property)->first()->id;
```
或者
```php
$this->select('id')->where('property', '=', property)->firstOrFail();
```

----

### 级联删除

> [Automatically deleting related rows in Laravel (Eloquent ORM)](http://stackoverflow.com/questions/14174070/automatically-deleting-related-rows-in-laravel-eloquent-orm)

在一对多的两个类的关系中，当一的一方被删除了之后，多的一方应该全部被删除
```php
class User extends Eloquent
{
    public function photos()
    {
        return $this->has_many('Photo');
    }

    // this is a recommended way to declare event handlers
    protected static function boot() {
        parent::boot();

        static::deleting(function($user) { // before delete() method call this
             $user->photos()->delete();
             // do the rest of the cleanup...
        });
    }
}
```
