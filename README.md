# 使用nodejs+express完成简单的文件上传功能

---

1.建立express项目
```
express -e upload
```
2.下载multer中间件
```
npm i multer or yarn multer
```
3.在routes/index.js中引用multer，由于还要使用到文件操作，还要引用fs模块,并指定文件上传目录
```
const multer  = require('multer');
const fs = require('fs');

const UPLOAD_PATH = './uploads'
```

`单文件上传: ` index.html中文件如下(*`form文件的类型必须为 enctype="multipart/form-data"`*), 
```
<form action="/upload" method="post" enctype="multipart/form-data">
    <input name="fileUpload" type="file" />
    <img src="" alt="">
    <button type="submit">上传</button>
</form>
```

在routes/index.js里面添加upload路由用来处理上传文件的操作
```
router.post('/upload', upload.single('fileUpload'), function (req, res, next) {
  const { file } = req;
  fs.readFile(file.path, function(err, data) {
    fs.writeFile(`${UPLOAD_PATH}/${file.originalname}`, data, function (err) {
      if (err) res.json({ err })
      res.json({
        msg: '上传成功'
      });
    });
  })
})
```

如果上传的文件是图片的话，还可以使用FileReader对象实现图片预览
```
(function($){
  $('input').on('change', function(event) {
    var files = event.target.files,
        reader = new FileReader();
    if(files && files[0]){
        reader.onload = function (ev) {
            $('img').attr('src', ev.target.result);
        }
        reader.readAsDataURL(files[0]);
    }
  })
}(jQuery))
```


`多文件上传:` 多文件上传的原理和单文件上传的原理一样，代码如下：
```
router.post('/upload', upload.array('fileUpload'), function (req, res, next) {
  const files  = req.files;
  const response = [];
  const result = new Promise((resolve, reject) => {
    files.map((v) => {
      fs.readFile(v.path, function(err, data) {
        fs.writeFile(`${UPLOAD_PATH}/${v.originalname}`, data, function(err, data) {
          const result = {
            file: v,
          }
          if (err)  reject(err);
          resolve('成功');
        })
      })
    })
  })
  result.then(r => {
    res.json({
      msg: '上传成功',
    })
  }).catch(err => {
    res.json({ err })
  });
})
```