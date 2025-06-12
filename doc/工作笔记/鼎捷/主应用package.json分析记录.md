---
tags:
  - 工作笔记/鼎捷/备忘
---
#### @webdpt/framework 4.1.0-1011
##### 使用到的功能点：
>[!Note] DwLanguageService **国际化相关**
>使用的能力包括：
>currentLanguage 当前语种
>language$ 监听语种变化
>translate 指令 ？？？ DwLanguageService和LanguageService都提供了这个指令，而且都在项目中引入了,且我没有找到初始化@ngx-translate/core的地方，那么大概率DwLanguageService对@ngx-translate/core进行了初始化，那么重写能力的时候需要注意

>[!Note] DwSystemConfigService **系统配置信息相关**
>应该是App启动之前，读取api.dev.json/api.json装载在全局的系统配置信息

>[!Note] DwUserService **登陆用户信息相关**
>项目中大量使用了getUser方法，但是并没有明确的设置用户信息的地方，大概率是DwSsoService内部做了这个事情，那么怎么做且用户信息长什么样需要测试

#### 全局变量
>[!Info] 
>通过FrameworkModule初始化，参数为常量SYSTEM_CONFIG，生成的InjecttionToken

>[!Note] 
>1. DW_APP_AUTH_TOKEN 应用token
>2. [[主应用package.json分析记录#^f8b0a0|DW_AUTH_TOKEN]]
>3. DW_USING_TAB **是否启用多页布局**
>4. DW_APP_ID **应用ID**
>5. Logo_Path **logo路径**
>6. LONIG_DEFAULT **登录页路径**

#### 登陆&鉴权相关
>[!Note] 文件地址：/athena_designer_web/apps/main/src/pages/login/sso/sso.service.ts
>> 未实际使用，传入了DwSsoService的构造函数,DwSsoService继承自SsoService，SsoService由SsoLoginModule使用，/sso-login路由会加载这个module，所以sso-login路由是干嘛的。。。看起来类似微信扫码登陆，token是urlParams带过来的。当然，DwSsoService重写了SsoService的ssoLogin方法，且内部并没有使用基类的方法，所以可以直接删除
>1. DwAuthService
>2. DwTenantService
>3. DwSsoService
>4. DwIamRepository
>5. DwSsoTokenRefreshService
>
#### 其它

>[!Note] IDWDmcUserInfo
>IAppConfig的类型声明中dwDmcUserInfo的类型为IDWDmcUserInfo，使用只有一处：
/athena_designer_web/apps/main/src/app/app.module.ts：SYSTEM_CONFIG，这个常量是@webdpt/framework包的FrameworkModule初始化时使用的

>[!Note] DwIamModule
>文档没有
/athena_designer_web/apps/main/src/app/app.module.ts中在NgModule中声明时import进去了，没有文档，不知道用没用，可以删了看是否报错

>[!Note] SessionStorage
>/athena_designer_web/apps/main/src/pages/login/service/user-storage.ts
只有这边用定义了一个AdUserStorage继承自SessionStorage
/athena_designer_web/apps/main/src/pages/login/service/user.service.ts
这个service使用了AdUserStorage，并大量被调用
所以SessionStorage干了什么特别的吗？？？

>[!Note] IDwSsoLogin
>/athena_designer_web/apps/main/src/pages/login/sso/sso-login.component.ts
作为类型被使用

>[!Note] DwProgramExecuteService
>/athena_designer_web/apps/main/src/pages/login/sso/sso-login.component.ts
/athena_designer_web/apps/main/projects/lcdp/src/lib/app/lowcode-progran-execute.service.ts
两处调用分别使用了这个service的方法：
this.dwProgramExecuteService.byId(programId, newParams);
this.dwProgramExecuteService.executeTabProgram$.next(routeInfo); 这边并没有接受消息的地方

#### @webdpt/iv-viewer 2.0.1
> 暂时没发现使用的地方，但是思伟说这块被webdpt/framework强依赖，待测试

#### @athena/design-ui 2.2.19-1024
1. 样式
>angular.json中./node_modules/@athena/design-ui/src/styles/index.less需要替换到ng-zorro的全局样式
且各组件样式需要保持一致
2. 组件
	1. ath-spin -> nz-spin
	2. ath-input-group -> nz-input-group
	3. ath-tree-select -> nz-tree-select
	4. ath-tree -> nz-tree
	5. ath-modal -> nz-modal
	6. ath-select -> nz-select, ath-option -> nz-option, ath-option-group -> nz-option-group
	7. ath-input-number -> nz-input-number
	8. ath-empty -> nz-empty
	9. ath-tag -> nz-tag
	10. ath-tabs -> nz-tabs, ath-tab -> nz-tab
	11. ath-radio-group -> nz-radio-group, ath-radio -> nz-radio
	12. ath-checkbox-group -> nz-checkbox-group, ath-checbox -> nz-checkbox
	13. ath-date-picker -> nz-date-picker
	14. ath-switch -> nz-switch
	15. ath-drawer -> nz-drawer
	16. ath-range-picker -> nz-range-picker
	17. ath-steps -> nz-steps, ath-step -> nz-step
	18. ath-button -> nz-button
#### 额外说明
>[!Tip] DW_AUTH_TOKEN
>这个值并不在常量中，什么地方什么时候设置不知道，但是在
>**/athena_designer_web/apps/main/src/pages/login/sso/sso.service.ts**中有这么一段：
>```javascript
>// 調用 DwIamHttpClient 時, 會取出 DW_AUTH_TOKEN, 所以要先設定
>this.adAuthService.setAuthToken({ iamToken: userToken });
>```
>但是很明显AdAuthService中的这个方法如下：
>```javascript
>/**
>  * 設定 AD_AUTH_TOKEN 的值提供給外部使用
>  *
>*/
>setAuthToken(datas: any): void {
>  if (Object.keys(datas).length > 0) {
>    this.userService.setUserInfo(datas);
>  }
>  this.authToken.token = this.userService.getUser('token') ?this.userService.getUser('token') : '';
>  this.authToken.iamToken = this.userService.getUser('iamToken') ? this.userService.getUser('iamToken') : '';
>}
>```
>所以这边是逻辑有bug还是说AD_AUTH_TOKEN和DW_AUTH_TOKEN是同一个？？又或者是打通的？？

^f8b0a0

