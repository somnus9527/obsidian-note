---
tags:
  - 知识笔记/经验
---
>[!Info] 备注
>**添加规则文件可帮助模型精准理解你的编码偏好，如框架、代码风格等**
**规则文件只对当前工程生效，单文件限制10000字符。如果无需将该文件提交到远程 Git 仓库，请将其添加到 .gitignore**

你是一位资深的TypeScript前端工程师，严格遵循DRY/KISS原则，精通响应式设计模式，注重代码可维护性与可测试性，遵循Airbnb TypeScript代码规范，熟悉React/Vue/Angular等主流框架的最佳实践。

## 技术栈规范：

- **框架**：Angular 12 + TypeScript

- **状态管理**：angular原生状态管理 Injectable Service + RxJs

- **路由**：@angular/router

- **HTTP请求**：Axios + 自定义API服务封装

- **测试**：Jest + e2e

- **构建工具**：@angular-builders/custom-webpack:browser

- **代码规范**：ESLint + Prettier + Husky预提交检查

## 应用逻辑设计规范：

### 1. 组件设计规范

#### 基础原则：

- 所有UI组件必须严格遵循单职责原则（SRP）

- 容器组件与UI组件必须分离（Presentational/Container模式）


#### 开发规则：

1. 组件必须严格遵守angular规范，独立封装，不能有全局状态

2. 组件必须严格安装xx.component.ts,xx.component.html,xx.component.less,xx.component.service.ts等文件
  - 如果组件内部需要拆分更原子的组件，则在该component文件夹中增加components文件夹，并在其中开发更原子的组件
  - 如果组件内部需要拆出工具方法
    - 如果工具方法相对简单，则直接在component根目录新增xx.component.tools.ts文件
    - 如果工具方法相对复杂，则直接在component根目录新增tools文件夹，并在其中开发工具方法
  - 如果组件内部需要拆出类型申明文件
    - 如果工具方法相对简单，则直接在component根目录新增xx.component.types.ts文件
    - 如果工具方法相对复杂，则直接在component根目录新增types文件夹，并在其中开发工具方法
  - 同工具方法和申明文件的操作方式，样式文件，service文件也按同样的逻辑处理，简单则使用单一文件，复杂则拆一个styles或者service文件夹

3. 避免使用`any`类型，必须明确标注类型

4. 状态管理独立到xx.service.ts中

5. 消息通信必须使用RxJs进行传递

6. 原子组件优先使用src/components/ad-ui-components中的组件，特殊情况使用ng-zorro-antd

7. 图标统一使用ad-icon组件，图标名称使用src/assets/icons/iconfont.js中的图标名称

8. directive优先使用src/common/directive内部定义的directive，没有则在改文件夹中申明新的directive

9. pipe优先是有src/common/pipe内部定义的pipe，没有则在改文件夹中申明新的pipe

## 代码规范细则：

### 1. 类型系统规范

- 必须使用接口（interface）定义类型

- 禁止使用`any`类型，必须明确标注`unknown`并做类型守卫

- 联合类型必须使用`|`明确标注

- 泛型使用必须标注约束条件

### 3. 代码风格规范

1. 必须使用PascalCase命名组件

2. 函数/变量名必须使用camelCase

3. 接口/类型名必须使用PascalCase

4. 常量必须使用UPPER_CASE

5. 禁止使用`console.log`提交代码

6. 必须使用TypeScript严格模式（`strict: true`）

### 4.核心代码模板示例

1. 模版文件
```html
<!--action列表开窗-->
<ad-modal
  nzClassName="app-action-list-modal"
  [nzWidth]="'800px'"
  [nzTitle]="'dj-选择action' | translate"
  [(nzVisible)]="transferModal"
  [nzFooter]="null"
  [nzClosable]="true"
  [nzMaskClosable]="false"
  (nzOnCancel)="handleCloseSelect()"
>
  <ng-container *adModalContent>
    <ad-tabs
      style="margin-bottom: 20px"
      [navStyle]="'default'"
      [(nzSelectedIndex)]="selectedIndex"
      (nzSelectedIndexChange)="handleSelectedIndexChange($event)"
    >
      <ad-tab *ngFor="let tab of tabs" [nzTitle]="tab.label | translate"></ad-tab>
    </ad-tabs>
    <div nz-row [nzGutter]="24">
      <div nz-col [nzSpan]="12" class="actionType" *ngIf="showActionType()">
        <span>
          {{ 'dj-action类型' | translate }}
        </span>
        <ad-select
          [nzPlaceHolder]="'dj-请选择' | translate"
          [(ngModel)]="labelType"
          (ngModelChange)="handleChangeType($event)"
        >
          <ng-container *ngFor="let data of actionTypeData">
            <ad-option [nzValue]="data.labelType" [nzLabel]="data.labelName"></ad-option>
          </ng-container>
        </ad-select>
      </div>
      <div nz-col [nzSpan]="12" class="searchAction">
        <nz-input-group class="search-input" [nzSuffix]="suffixIconSearch">
          <input
            type="text"
            nz-input
            [placeholder]="'dj-请输入关键字' | translate"
            [(ngModel)]="searchValue"
            (keyup.enter)="handleSearchAction()"
          />
        </nz-input-group>
        <ng-template #suffixIconSearch>
          <i adIcon type="search" (click)="handleSearchAction()"></i>
        </ng-template>
      </div>
    </div>
    <div class="attr-table">
      <ad-table
        style="width: 100%; height: 400px"
        [headerHeight]="40"
        [loading]="loading"
        [rowData]="allActions"
        [columnDefs]="columnDefs"
        [frameworkComponents]="[]"
        [rowCheckbox]="true"
        [rowSelection]="'single'"
        checkboxBindField="checked"
        [rowBuffer]="20"
        [pagination]="true"
        [frontPagination]="false"
        [showPaginationTotal]="true"
        [total]="actionTotal"
        [pageNumber]="actionPageNum"
        [pageSize]="actionPageSize"
        [showSizeChanger]="true"
        (rowSelected)="handleRowSelected($event)"
        (paginationChanged)="handlePaginationChanged($event)"
        (firstDataRendered)="onFirstDataRendered($event)"
      >
      </ad-table>
      <!-- <div class="page-number" *ngIf="allActions?.length > 0">100 {{ 'dj-条/页' | translate }}</div> -->
    </div>
    <div *ngIf="allActions?.length > 0 && !selectedActionId" class="red-tip">
      {{ 'dj-请选择一项数据' | translate }}
    </div>
    <div class="modal-footer">
      <button ad-button adType="default" (click)="handleCloseSelect()">
        {{ 'dj-取消' | translate }}
      </button>
      <button ad-button adType="primary" (click)="handleChangeAction()">
        {{ 'dj-确定' | translate }}
      </button>
    </div>
  </ng-container>
</ad-modal>
```

2. 样式文件
```less
::ng-deep .app-action-list-modal {
  .actionType {
    .ant-select {
      width: 200px;
      margin-left: 10px;
    }
  }
  .searchAction {
    width: 50%;
    margin-bottom: 10px;
  }
  .searchAction {
    margin-bottom: 10px;
  }
  .attr-table {
    height: 444px;
    position: relative;
    ::ng-deep {
      ad-table {
        // .ag-cell {
        //   line-height: 34px;
        // }
        // .athena-table-pagination {
        //   margin-top: 4px;
        // }
      }
    }
    .page-number {
      position: absolute;
      left: 0;
      top: 419px;
      font-size: 12px;
    }
  }
  .red-tip {
    color: #ea3d46;
    font-size: 13px;
  }
  .modal-footer {
    text-align: center;
    justify-content: center;
    padding-top: 30px;
    border: 0;
    button {
      width: 88px;
      height: 32px;
      border-radius: 20px;
      margin: 0 11px;

      &:not(.ant-btn-primary) {
        border-color: #6a4cff;
        color: #6a4cff;

        &:hover {
          border-color: #5a4fee;
          color: #5a4fee;
        }
      }
    }
  }
}
```

3. 组件文件
```typescript
import { ChangeDetectorRef, Component, EventEmitter, Input, OnInit, Output, SimpleChanges } from '@angular/core';
import { AppService } from '../../../pages/apps/app.service';
import { TranslateService } from '@ngx-translate/core';
import { NzMessageService } from 'ng-zorro-antd/message';
import { AdModalService } from 'components/ad-ui-components/ad-modal/ad-modal.service';
import { ActionModalService } from './action-modal.service';
import { isNotNone } from 'common/utils/core.utils';

@Component({
  selector: 'app-action-modal',
  templateUrl: './action-modal.component.html',
  styleUrls: ['./action-modal.component.less'],
  providers: [ActionModalService],
})
export class ActionModalComponent implements OnInit {
  @Input() transferModal: boolean;
  @Input() transferData: any; // selectType: 是否显示type
  @Input() confirmTip: any;
  @Input() labelType: any;
  @Input() favouriteCode: string;
  @Input() applicationCodeProxy: string;

  @Output() callBack = new EventEmitter();
  @Output() closeModal = new EventEmitter();

  loading: boolean;
  searchValue: any; // 搜索关键字
  actionTypeData: any; // action type数据
  allActions: any[] = []; // 所有action数据

  selectedActionId: any; // 选中actionId
  selectedActionName: any; // 选择actionName
  selectedActionType: any; // 选择action类型
  selectedApplication: any; // 选择action所属解决方案

  actionPageNum: any; // action列表页码
  actionPageSize: number = 100; // action列表步长
  actionTotal: any; // action总数

  columnDefs: any[];

  selectedIndex: number = 0;
  tabs: any[] = [
    { label: 'dj-当前应用', value: 'curApp' },
    { label: 'dj-应用共享', value: 'appShare' },
    { label: 'dj-全局', value: 'global' },
  ];

  constructor(
    public actionService: ActionModalService,
    public service: AppService,
    private translate: TranslateService,
    private message: NzMessageService,
    private modal: AdModalService,
    private cd: ChangeDetectorRef,
  ) {}

  ngOnInit(): void {
    this.handleInitTabs();
    this.columnDefs = [
      {
        headerName: 'actionId',
        field: 'actionId',
        filter: false,
        sortable: false,
        width: 150,
        tooltipValueGetter: (params) => {
          return params.value || ' ';
        },
      },
      {
        headerName: 'actionName',
        field: 'actionName',
        filter: false,
        sortable: false,
        width: 90,
        tooltipValueGetter: (params) => {
          return params.value || ' ';
        },
      },
      {
        headerName: 'label',
        field: 'label',
        filter: false,
        sortable: false,
        width: 60,
        tooltipValueGetter: (params) => {
          return params.value || ' ';
        },
      },
    ];
    this.handleInit();
  }

  ngOnChanges(changes: SimpleChanges): void {
    if (changes.favouriteCode?.currentValue) {
      this.actionService.favouriteCode = this.favouriteCode;
      this.actionService.headerCofig = { templateId: this.favouriteCode };
    }
    if (changes.applicationCodeProxy?.currentValue) {
      this.actionService.applicationCodeProxy = this.applicationCodeProxy;
    }
  }

  // 选择action开窗
  handleInit(): void {
    if (!!this.transferData['selectType']) {
      this.handleLoadActionType();
    } else {
      this.searchValue = '';
      this.selectedActionId = this.transferData['actionId'];
      this.selectedActionName = this.transferData['actionName'];
      this.selectedActionType = this.transferData['actionType'];
      let param;
      if (!this.labelType) {
        param = {
          condition: this.searchValue || '',
          pageNum: this.actionPageNum || 1,
        };
      } else {
        param = {
          condition: this.searchValue || '',
          pageNum: this.actionPageNum || 1,
          labelType: this.labelType,
        };
      }
      this.handleLoadAction(param);
    }
  }

  /**
   * 初始化tab标签
   */
  handleInitTabs(): void {
    if (!this.showGlobalTab()) {
      this.tabs.splice(-1, 1);
    }
  }

  // 左侧: 查询action类型
  handleLoadActionType(): void {
    if (this.transferData?.selectOptions) {
      this.actionTypeData = [...this.transferData?.selectOptions];
    } else {
      this.actionService.loadActionType().subscribe((res) => {
        if (res.code === 0) {
          this.actionTypeData = res.data || [];
        }
      });
    }
    this.handleChangeType(this.labelType);
  }

  // 修改action类型
  handleChangeType(val: any): void {
    this.actionPageNum = 1;
    const param: any = {
      labelType: val,
      condition: this.searchValue || '',
      pageNum: this.actionPageNum,
    };
    this.handleLoadAction(param);
  }

  handlePaginationChanged(event) {
    if (event.type === 'page-number') {
      this.actionPageNum = event.value;
      this.handleChangeActionNumOrSize();
    } else if (event.type === 'page-size') {
      this.actionPageSize = event.value;
      this.handleChangeActionNumOrSize();
    }
  }

  // action分页
  handleChangeActionNumOrSize(): void {
    let param;
    if (!this.labelType) {
      param = {
        condition: this.searchValue || '',
        pageNum: this.actionPageNum,
      };
    } else {
      param = {
        condition: this.searchValue || '',
        pageNum: this.actionPageNum,
        labelType: this.labelType,
      };
    }
    this.handleLoadAction(param);
  }

  // 模糊匹配
  handleSearchAction(): void {
    let param;
    if (!!this.searchValue) {
      this.actionPageNum = 1;
    }
    if (!this.labelType) {
      param = {
        condition: this.searchValue || '',
        pageNum: this.actionPageNum || 1,
      };
    } else {
      param = {
        condition: this.searchValue || '',
        pageNum: this.actionPageNum || 1,
        labelType: this.labelType,
      };
    }
    this.handleLoadAction(param);
  }

  // 查询action
  handleLoadAction(paramStr: any): void {
    this.loading = true;
    let params: any = {
      type: this.tabs[this.selectedIndex]?.value,
      condition: paramStr?.condition,
      pageNum: paramStr?.pageNum,
      pageSize: this.actionPageSize,
      needEsp: this.showGlobalTab(),
    };
    if (this.showActionType() && isNotNone(this.labelType)) {
      params = {
        ...params,
        label: this.labelType,
      };
    }

    if (this.service.selectedApp?.code) {
      params = {
        ...params,
        appCode: this.service.selectedApp?.code,
      };
    }

    this.actionService.loadActions(params).subscribe(
      (res) => {
        this.loading = false;
        if (res.code === 0 && params.type === this.tabs[this.selectedIndex]?.value) {
          this.allActions = res.data?.data || [];
          this.actionTotal = res.data?.total || 0;
          this.actionPageNum = res.data?.curPageNum || 0;
          this.allActions.map((item) => {
            item.checked = this.selectedActionId === item.actionId;
            return item;
          });
          // setTimeout(() => {
          //   const child = document.querySelector('.ant-pagination')?.children;
          //   const dom = document.querySelector('.page-number');
          //   if (!!dom) {
          //     dom['style'].left = `${child?.[0]?.['offsetLeft'] - 60}px`;
          //   }
          //   this.cd.detectChanges();
          // }, 10);
        }
        this.cd.detectChanges();
      },
      () => {
        this.loading = false;
      },
    );
  }

  handleRowSelected(event) {
    if (event.data?.checked) {
      this.handleSelectAction(event.data);
    }
  }

  // 选择action
  handleSelectAction(data: any): void {
    this.selectedActionId = data['actionId'];
    this.selectedActionName = data['actionName'];
    this.selectedActionType = data['label'];
    this.selectedApplication = data['application'];
  }

  // 关闭action
  handleCloseSelect(): void {
    this.selectedActionId = '';
    this.selectedActionName = '';
    this.selectedActionType = '';
    this.selectedApplication = '';
    this.actionTotal = 0;
    this.actionPageNum = 1;
    this.closeModal.emit();
  }

  // 确认action
  handleChangeAction(): void {
    if (!!this.selectedActionId) {
      const toDo = (): void => {
        this.actionTotal = 0;
        this.actionPageNum = 1;
        if (!!this.selectedActionId && !this.selectedActionType) {
          this.selectedActionType = (this.allActions.find((s) => s.actionId === this.selectedActionId) || {}).label;
        }
        this.callBack.emit({
          actionId: this.selectedActionId || '',
          actionName: this.selectedActionName || '',
          actionType: this.selectedActionType || '',
          application: this.selectedApplication || '',
        });
      };
      if (this.confirmTip) {
        this.modal.confirm({
          nzTitle: this.translate.instant('dj-确认修改action？'),
          nzWrapClassName: 'vertical-center-modal',
          nzOkText: this.translate.instant('dj-确定'),
          nzOnOk: () => {
            toDo();
          },
          nzOnCancel: () => {},
        });
      } else {
        toDo();
      }
    }
  }

  onFirstDataRendered(params) {
    params.api.sizeColumnsToFit();
  }

  handleSelectedIndexChange(e: any): void {
    this.searchValue = null;
    this.actionPageNum = 1;
    // this.actionPageSize = 100;
    if (this.showActionType()) {
      this.labelType = null;
    }
    this.handleSearchAction();
  }

  /**
   * 显示action类型
   * @returns
   */
  showActionType(): boolean {
    return [1, 2].includes(this.selectedIndex) && !!this.transferData['selectType'];
  }

  /**
   * 显示全局的Tab
   * @returns
   */
  showGlobalTab(): boolean {
    return !!this.transferData['selectType'] || this.labelType === 'EspAction';
  }
}
```

4. module定义

```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TranslateModule } from '@ngx-translate/core';
import { NzFormModule } from 'ng-zorro-antd/form';
import { ActionModalComponent } from './action-modal.component';
import { FormsModule } from '@angular/forms';
import { AdModalModule } from 'components/ad-ui-components/ad-modal/ad-modal.module';

import { NzInputModule } from 'ng-zorro-antd/input';
import { NzTableModule } from 'ng-zorro-antd/table';
import { AdButtonModule } from 'components/ad-ui-components/ad-button/ad-button.module';
import { NzRadioModule } from 'ng-zorro-antd/radio';
import { AdIconModule } from 'components/ad-ui-components/ad-icon/ad-icon.module';
import { AthenaTableModule } from '../athena-table/athena-table.module';
import { AdSelectModule } from 'components/ad-ui-components/ad-select/ad-select.module';
import { NzMessageModule } from 'ng-zorro-antd/message';
import { AdTabsModule } from 'components/ad-ui-components/ad-tabs/ad-tabs.module';

@NgModule({
  declarations: [ActionModalComponent],
  imports: [
    CommonModule,
    NzFormModule,
    FormsModule,
    AdModalModule,
    TranslateModule,
    NzMessageModule,
    AdSelectModule,
    NzInputModule,
    NzTableModule,
    NzRadioModule,
    AdButtonModule,
    AdIconModule,
    AthenaTableModule,
    AdTabsModule,
  ],
  exports: [ActionModalComponent],
})
export class ActionModalModule {}
```

5. service定义
```typescript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { SystemConfigService } from 'common/service/system-config.service';
import { Observable } from 'rxjs';

@Injectable()
export class ActionModalService {
  adesignerUrl: string;

  /* 以下用于模板管理： */
  headerCofig = {}; // 用于保存请求头
  favouriteCode = null; // 收藏code
  applicationCodeProxy = null; // 解决方案code

  constructor(protected http: HttpClient, protected configService: SystemConfigService) {
    this.configService.get('adesignerUrl').subscribe((url) => {
      this.adesignerUrl = url;
    });
  }

  // 私有: 获取action type数据
  loadActionType(): Observable<any> {
    const url = `${this.adesignerUrl}/athena-designer/action/findActionLabels`;
    return this.http.get(url);
  }

  // 查询action
  loadActions(params: any): Observable<any> {
    if (this.applicationCodeProxy && Reflect.has(params, 'appCode')) {
      params.appCode = this.applicationCodeProxy;
    }
    const url = `${this.adesignerUrl}/athena-designer/action/findActionsV2`;
    return this.http.post(url, params);
  }
}
```

6. directive定义
```typescript
import { Directive, EventEmitter, HostListener, Input, OnDestroy, OnInit, Output } from '@angular/core';
import { Subject, Subscription } from 'rxjs';
import { debounceTime } from 'rxjs/operators';

@Directive({
  selector: '[appDebounce]',
})
export class DebounceDirective implements OnInit, OnDestroy {
  @Input('appDebounceClick') debounceTime = 500;
  @Input('appDebounceInput') inputDebounceTime = 500;
  @Output() debounceClick = new EventEmitter();
  @Output() debounceInput = new EventEmitter();
  private clicks = new Subject<any>();
  private inputs = new Subject<any>();
  private subscription: Subscription;
  private inputSubscription: Subscription;

  constructor() {}

  ngOnInit() {
    this.subscription = this.clicks.pipe(debounceTime(this.debounceTime)).subscribe((e) => this.debounceClick.emit(e));
    this.inputSubscription = this.inputs
      .pipe(debounceTime(this.inputDebounceTime))
      .subscribe((e) => this.debounceInput.emit(e));
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
    this.inputSubscription.unsubscribe();
  }

  @HostListener('click', ['$event'])
  clickEvent(event) {
    event.preventDefault();
    event.stopPropagation();
    this.clicks.next(event);
  }

  @HostListener('input', ['$event'])
  inputEvent(event) {
    event.preventDefault();
    event.stopPropagation();
    this.inputs.next(event.srcElement['value']);
  }
}
```

## 代码提交规范

1. 必须通过Husky预提交检查

2. 提交信息必须符合`<type>: <subject> (<scope>)`格式（如`feat: 添加登录功能 (user)`）

3. 必须包含禅道任务号（如`#123`）

4. 必须通过Code Review后合并

```