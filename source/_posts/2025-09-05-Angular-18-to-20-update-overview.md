---
title: Angular, v18 到 v20 的新功能重點整理
date: 2025-09-05 01:01:30
updated: 2025-09-07
categories:
tags:
  - Angular
cover: https://i.imgur.com/FZBxAmF.png
---

接續 [上一篇(v15 到 v17)](/2025-09-01/Angular-15-to-17-update-overview/)，這篇將介紹 Angular v18 到 v20 中我整理的主要的新功能。

---

## Angular v18

### Zoneless change detection

- 在之前 Angular 依賴 zone.js 來觸發變更偵測，但 zone.js 在開發體驗及效能方面有一些缺點。Angular v18 開始推出「zoneless」的實驗性 API，讓開發者可以不再依賴 zone.js。
- 在 zoneless 模式下，官方建議在元件中使用 signals 來管理狀態，變更偵測將依據 signal 的更新來觸發。
- 使用 zoneless 範例:

  ```typescript
  // main.ts

  bootstrapApplication(App, {
    providers: [
      provideExperimentalZonelessChangeDetection()
    ]
  });
  ```

  ```json
  // angular.json
  
  "polyfills": [
    "zone.js" // remove this line if you are using zoneless mode
  ],
  ```

### 預設啟用 Coalescing

v18 開始，新專案無論是 zoneless 或 zone.js ，都將預設啟用 coalescing，其作用為將短時間內多次的變更偵測需求合併，減少不必要的變更以提升效能。

- 使用 zone.js 的舊專案可以新增 `eventCoalescing: true` 來啟用此功能。

  ```typescript
  // main.ts

  bootstrapApplication(App, {
    providers: [
      provideZoneChangeDetection({ eventCoalescing: true })
    ]
  });
  ```

### ng-content 支援預設內容

- 過去 ng-content 只能用來投影外部傳入的內容， v18 開始支援預設，當父元件沒有傳入內容時，就會顯示預設內容。

  ```typescript
  <ng-content select=".custom-content">
    <span>Here is the default content</span>
  </ng-content>
  ```

### AbstractControl 新增 event 屬性

- v18 開始為 `AbstractControl` 新增 `event` 屬性，它將回傳一個 observable，讓開發者可以更方便地監聽各種表單事件。

  ```typescript
  const control = new FormControl<string|null>('test', Validators.required);
  control.event.subscribe(event => {
    console.log('Form control event:', event);
  });
  ```

### Route `redirectTo` 支援函式

- 在 Angular v18，redirectTo 現在可以接受一個函式，讓你根據執行時狀態動態決定要導向哪個路徑。這樣可以實現更複雜的導向邏輯，例如根據登入狀態、權限等。

  ```typescript
  const routes: Routes = [
    {
      path: '',
      redirectTo: ({ queryParams }) => {
        const userId = queryParams['userId'];
        const isLoggedIn = !!userId && checkUserLoggedIn(userId);
        return isLoggedIn ? '/dashboard' : '/login';
      }
    }
  ];
  ```

### Event replay

- 在使用 SSR 且啟用 hydration 時，當頁面尚未完成「hydration」時，使用者的操作（例如點擊、輸入等）將不會有作用。
- v18 開始支援預覽版的 Event replay (事件重播)，Angular 會先記錄所有使用者事件，等到頁面完成 hydration 後再重播這些事件，確保不會漏掉這些互動。
- 使用 `withEventReplay()` 來啟用，範例:

  ```typescript
  // main.ts
  
  bootstrapApplication(App, {
    providers: [
      provideClientHydration(withEventReplay())
    ]
  });
  ```

### v18 其他更新

- @if 等新的控制流程語法進入穩定版
- @defer 等延遲載入語法進入穩定版
- Material 3 進入穩定版
  - 前面的版本推出了 Material 3 的實驗性支援，並在 v18 已正式升級為「穩定版」。
  - Angular Material 新版教學: https://material.angular.io/guide/theming
- HttpClientModule 棄用
  - 改用 `provideHttpClient()` 來取代。
  
    ```typescript
    bootstrapApplication(App, {
      providers: [provideHttpClient()]
    });
    ```

---

## Angular v19

### Incremental Hydration

- v19 新增 `Incremental Hydration`（增量式水合）的開發者預覽版，讓頁面可以分段進行 hydration，而不是一次性將整個頁面水合完成。這樣可以提升初始載入效能，讓使用者更快看到可互動的內容。例如，頁面一開始只有主選單活化，其他部分灰階(未水合)，等使用者滑到或點擊才載入對應元件將其「水合」。

- 使用 `withIncrementalHydration()` 來啟用，範例:

  ```typescript
  // main.ts

  bootstrapApplication(AppComponent, {
    providers: [provideClientHydration(withIncrementalHydration())]
  });
  ```

  ```html
  <!-- html -->

  <!-- 當使用者點擊 <promo-banner> 時才水化元件 -->
  @defer (hydrate on interaction(click)) {
    <promo-banner/>
  }
  ```

### ServerRoute 和 RenderMode

- 以往啟用 SSR（Server-Side Rendering）時，Angular 會有以下方式處理路由：

  - 針對沒有參數的路由進行 prerender，在建置階段，預先產生好 HTML 靜態檔案部署到伺服器上，使用者請求時直接回傳這些靜態檔案。
  - 針對有參數的路由進行 server-side rendering，伺服器在收到請求時，根據路由參數動態產生 HTML 回傳給使用者。

- 在 v19 中，可以用新的 `ServerRoute` 介面，針對每條路由指定渲染模式，包含：

  - Prerender（預渲染）
  - Server（伺服器端渲染）
  - Client（客戶端渲染）
  
  ```typescript
  const serverRouteConfig: ServerRoute[] = [
    { path: '/login', mode: RenderMode.Server },      // 伺服器端渲染
    { path: '/dashboard', mode: RenderMode.Client },  // 客戶端渲染
    { path: '/**', mode: RenderMode.Prerender },      // 其他路由預渲染
  ];
  ```

- 這主要解決了帶參數路由的預渲染需求，不需重複宣告路由；  
  也能針對高度動態、不需 SEO 的內容(儀表板、登入後的會員頁面等)，改用客戶端渲染以減少伺服器負擔。

### Hot Module Replacement (HMR)

- 過去使用 Angular 開發時，每次修改元件樣式或模板並儲存檔案後，Angular CLI 都會重建整個應用程式並刷新瀏覽器，且導致應用狀態（例如表單輸入、滾動位置）會重置。

- v19 新增的 HMR 能直接編譯你修改的樣式或模板，只把變更部分即時套用到瀏覽器，無需整頁刷新，也不會丟失狀態。這可以大幅提升開發效率及體驗。

  - 樣式(style) 的 HMR 預設啟用。
  - 模板(template) 的 HMR 在 v19 中屬於實驗功能，需要用環境變數啟用：
  
    ```bash
    NG_HMR_TEMPLATES=1 ng serve
    ```

  - 關閉 HMR 可用 `--no-hmr` 參數：

    ```bash
    ng serve --no-hmr
    ```

### v19 其他更新

- Standalone 預設啟用；可以使用以下設定來強制使用 standalone 元件：

  ```json
  "angularCompilerOptions": {
    "strictStandalone": true
  }
  ```

- Angular CLI 會對 standalone 中未使用的 imports 發出警告(unusedStandaloneImports)。

- `@let` 語法進入穩定版，它允許在 template 裡能直接宣告區域變數。

  ```html
  <input #name>
  @let greeting = 'Hello ' + name.value;
  ```

- 新專案的 Event replay 將預設啟用

---

## Angular v20

### 新的實驗性 resource 相關 API

Angular v19 開始推出 resource API，v20 增加 resource streaming 和 httpResource，
目的是用 Signal-based reactive API 更方便管理非同步狀態。

#### resource API

- 讓你在 signal 改變時，啟動非同步行為（如請求資料），並將結果以 signal 形式暴露。
- 主要用法：  

  ```typescript
  const userId: Signal<string> = getUserId();
  const userResource = resource({
    params: () => ({id: userId()}),
    loader: ({request, abortSignal}): Promise<User> => {
      return fetch(`users/${request.id}`, {signal: abortSignal});
    },
  });
  ```

  - 當 `userId` signal 改變時，會自動重新發起請求。

#### Streaming Resource

- 用於資料流（例如 WebSocket），可持續接收數據。
- 範例用法：
  
  ```typescript
  @Component({
    template: `{{ dataStream.value() }}`
  })
  export class App {
    dataStream = resource({
      stream: () => {
        return new Promise<Signal<ResourceStreamItem<string[]>>>(resolve => {
          const resourceResult = signal<{ value: string[] }>({ value: [] });
          this.socket.onmessage = event => {
            resourceResult.update(current => ({
              value: [...current.value, event.data]
            }));
          };
          resolve(resourceResult);
        });
      },
    });
  }
  ```

  - `resourceResult` 透過 signal 持續更新 WebSocket 收到的資料。

#### httpResource

- 以更簡潔的 signal API 進行 HTTP 請求。
- 範例用法：
  
  ```typescript
  @Component({
    template: `{{ userResource.value() | json }}`
  })
  class UserProfile {
    userId = signal(1);
    userResource = httpResource<User>(() => 
      `https://example.com/v1/users/${this.userId()}`
    );
  }
  ```
  
  - `userResource` 會隨著 `userId` signal 變化，重新發送 GET 請求。
  - 提供如 `isLoading`、`headers` 等屬性。

- 可透過 `HttpClient` 設定攔截器：
  
  ```typescript
  bootstrapApplication(AppComponent, {providers: [
    provideHttpClient(
      withInterceptors([loggingInterceptor, cachingInterceptor]),
    )
  ]});
  ```

### 動態元件創建與綁定

- 之前可使用 `createComponent` 動態建立元件，v20 新增可直接套用指令與指定綁定功能。
- 支援 signal-based 綁定、雙向綁定。
- 範例：
  
  ```typescript
  import {createComponent, signal, inputBinding, outputBinding} from '@angular/core';

  const canClose = signal(false);
  const title = signal('My dialog title');

  createComponent(MyDialog, {
    bindings: [
      inputBinding('canClose', canClose), // input 綁定 signal
      outputBinding<Result>('onClose', result => console.log(result)), // output 綁定 callback
      twoWayBinding('title', title), // 雙向綁定
    ],
    directives: [
      FocusTrap, // 套用指令
      {
        type: HasColor,
        bindings: [inputBinding('color', () => 'red')] // 指令綁定 input
      }
    ]
  });
  ```

  - 可將 signal 綁定到 input、output 綁定 callback、雙向綁定 signal。
  - 可直接套用 directives 並指定 input 綁定。

### 檔名和元件後綴變成「選擇性」

- **Angular CLI** 從 v20 起，預設不會為元件、指令、服務等產生後綴。
- 舊專案透過 `ng update` 會在 `angular.json` 啟用後綴產生。
- 鼓勵開發者更有意識地命名抽象，減少樣板程式碼。
- 若新專案需要產生後綴，可於 `angular.json` 設定 schematic：

  ```json
  {
    "projects": {
      "app": {
        "schematics": {
          "@schematics/angular:component": { "type": "component" },
          "@schematics/angular:directive": { "type": "directive" },
          "@schematics/angular:service": { "type": "service" },
          "@schematics/angular:guard": { "typeSeparator": "." },
          "@schematics/angular:interceptor": { "typeSeparator": "." },
          "@schematics/angular:module": { "typeSeparator": "." },
          "@schematics/angular:pipe": { "typeSeparator": "." },
          "@schematics/angular:resolver": { "typeSeparator": "." }
        },
      }
    }
  }
  ```

### v20 其他更新

- SSR 的 `incremental hydration` 和 `route-level rendering mode` 進入穩定版
- 擴充模板表達式語法: `**` 與 `in` 運算子
  
  ```html
  <!-- n 的平方 -->
  {{ n ** 2 }}

  <!-- 判斷 person 是否有 name 屬性 -->
  {{ name in person }}
  ```

- 為應對 Karma 的棄用，Angular CLI 實驗性的引入了 vitest 測試。

---

## 小結

整體來說，Angular 近幾年迎來了翻天覆地的變化，像是 zone.js 原本是舊專案變化檢測的核心，但現在卻被 zoneless 的方式逐漸取代，並且推薦使用 Standalone 和 Signals 來管理狀態。對舊專案進行升級難免會遇到一些轉換成本，但這些改動也確實提升了效能與開發體驗。而且官方也有提供 [升級指南](https://angular.dev/update-guide)，因此如果手上還有舊專案，還是建議可以在有餘力時升級到最新穩定版本喔。

---

### 參考來源

- [Angular 官方文件](https://angular.dev/)
- [Angular Blog](https://blog.angular.io/)
- [Angular 更新日誌](https://github.com/angular/angular/blob/main/CHANGELOG.md)
