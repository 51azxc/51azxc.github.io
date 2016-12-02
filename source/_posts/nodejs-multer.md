title: 解决multer无法识别ng-file-upload批量上传文件
date: 2016-04-23 22:56:28
tags: "multer"
categories: ["nodejs", "express"]
---

> [Multer not accepting files in array format gives 'Unexpected Filed Error'](http://stackoverflow.com/questions/32917617/multer-not-accepting-files-in-array-format-gives-unexpected-filed-error)

`ng-file-upload`上传插件批量上传文件时默认使用`files[0]`, `files[1]`, `files[2]`...这样的数组形式标识上传文件，而`multer`无法识别同样名字的上传文件。因此需要在`ng-file-upload`的配置中修改**arrayKey**属性：
```js
Upload.upload({
  url: '/upload',
  arrayKey: '', // default is '[i]'
  data: {
    files: files
  }
})
```
