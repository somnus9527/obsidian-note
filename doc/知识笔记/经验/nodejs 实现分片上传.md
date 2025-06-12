---
tags:
  - 知识笔记/经验
---
>[!Note] 依赖
>- @koa/router: ^10.0.0
>- koa: ^2.13.1
>- koa-bodyparser: ^4.3.0
>- multiparty: ^4.2.2
>- uuid: ^8.3.2

>[!Info] 实现逻辑:
>- 客户端先通过上传的先行接口向服务器获取一个唯一的hash_id,保证第三步正常合并,不会串分片
>- 接着上传分片,这里客户端有分片的处理,上传时需要带上hash_id
>- 最后客户端上传完所有分片,调用合并接口.服务端得到合并信息,开始针对该hash_id分片进行合并操作


> [!Info] 代码实现：
> ```javascript
> /// 解析formData
> const parseFormData = request => {
>     return new Promise((resolve,reject) => {
>         const form = new multiparty.Form();
>         form.parse(request,(error,fields,files) => {
>             const keys = Object.keys(fields);
>             let field = {};
>             keys.map(key => {
>                 const value = fields[key];
>                 field[key] = value[0];
>             })
>             if(error)
>                 reject(error);
>             resolve({
>                 fields: field,
>                 file: files.file[0]
>             })
>         })
>     })
> }
> /// 分片合并
> const pipeMerge = (index,files,inputPath,writeStream,cb) => {
>     const readStream = fs.createReadStream(`${inputPath}/${files[index]}`);
>     readStream.on('end', () => {
>         index++;
>         readStream.destory();
>         if(index < files.length)
>             pipeMerge(index,files,inputPath,writeStream,cb)
>         else
>             cb();
>     });
>     readStream.pipe(writeStream,{end: index === files.length - 1 ? true : false});
> }
> /// 将分片写入可写流
> const pieceFileMerge = (inputPath,outputPath) => {
>     return new Promise((resolve,reject) => {
>         try {
>             const files = fs.readdirSync(inputPath);
>             if(files.length <= 0) {
>                 return reject(new Error('找不到分片文件'));
>             }
>             const writeStream = fs.createWriteStream(outputPath);
>             pipeMerge(0,files,inputPath,writeStream,() => {
>                 resolve();
>             })
>         } catch (error) {
>             reject(error);
>         }
>     })
> }
> 
> router.post('beforePieceUpload',async (ctx,next) => {
>     const { userId , fileName } = ctx.request.body;
>     const hashPath = uuid.v4(`${userId}_${fileName}`);
>     ctx.body = {
>         "hash": hashPath
>     }
> })
> 
> router.post('pieceUpload',async (ctx,next) => {
>     const result = await parseFormData(ctx.req);
>     const { fields: { hash , userId , pieceNo } , file } = result;
>     const basePath = './upload_pack';
>     const hashPath = `${basePath}/${hash}_${userId}/`;
>     const hashFileName = `${hashPath}${hash}_${userId}_${pieceNo}`;
>     if(!fs.existsSync(basePath)) {
>         fs.mkdirSync(basePath);
>         fs.mkdirSync(hashPath)
>     } else if(!fs.existsSync(hashPath)) {
>         fs.mkdirSync(hashPath);
>     }
>     const fileName = path.resolve(__dirname,hashFileName);
>     await pipeline(
>         fs.createReadStream(file.path),
>         fs.createWriteStream(fileName),
>     )
>     ctx.body = {
>         "message": `piece ${pieceNo} upload success`,
>     }
> })
> 
> router.post('pieceUploadMerge',async (ctx,next) => {
>     const { userId , hash , fileType } = ctx.request.body;
>     const filePath = path.resolve(__dirname,`./upload_pack/${hash}_${userId}`);
>     await pieceFileMerge(filePath,`${filePath}/${hash}_${userId}.${fileType}`);
>     ctx.body = {
>         "message": "上传成功"
>     }
> })
> ```