---
title: Angular, v15 到 v17 的新功能重點整理
date: 2025-09-01 20:19:00
updated: 2025-09-07
categories:
tags:
  - Angular
cover: https://i.imgur.com/AC09hUs.png
---

記得幾年前我接觸 Angular 時，當時的版本還是 v15 。如今 Angular 已經進入 v20 版本了！這些版本中有許多新功能，讓我們來快速回顧一下重點更新，當然若須深入了解，官方文件有各功能更詳細的介紹可參考。

由於內容稍多，所以將分為上下篇；此篇為上篇，介紹 v15 到 v17 的主要新功能。 ( v18 到 v20 請見[下一篇](/2025-09-05/Angular-18-to-20-update-overview/))

---

## Angular v15

### Standalone Components 正式版

- 告別 NgModule，可以直接宣告元件(Component)、指令(Directive)、管道(Pipe)。

  ```typescript
  import { Component } from '@angular/core';

  @Component({
    standalone: true,
    selector: 'app-hello',
    template: `<h1>Hello Standalone!</h1>`,
  })
  export class HelloComponent {}
  ```

### Directive Composition API

- 元件可直接複用多個指令的行為，這個功能僅適用於 standalone 指令。
- 在 v15 為實驗性功能，後續版本已進入穩定版

  ```typescript
  // 定義可複用的指令
  @Directive({
    standalone: true,
    selector: '[hasColor]'
  })
  export class HasColor {
    @Input() color: string = '';
  }

  @Directive({
    standalone: true,
    selector: '[cdkMenu]'
  })
  export class CdkMenu {
    @Input() cdkMenuDisabled: boolean = false;
    @Output() cdkMenuClosed = new EventEmitter();
  }

  // 在元件中組合多個指令
  @Component({
    selector: 'mat-menu',
    standalone: true,
    hostDirectives: [
      HasColor,
      {
        directive: CdkMenu,
        inputs: ['cdkMenuDisabled: disabled'],
        outputs: ['cdkMenuClosed: closed']
      }
    ],
    template: `<div>Custom Menu</div>`
  })
  export class MatMenu {}
  ```

---

## Angular v16

### Signals

一種新的響應式狀態管理方式。

- 提供宣告式、可預測的資料流和變更偵測。
- 在 v16 為實驗性功能，後續版本已進入穩定版
- 官方詳細介紹: https://angular.dev/guide/signals

  ```typescript
  import { signal, computed, effect } from '@angular/core';

  interface User { name: string; age: number; }

  export class CounterComponent {
    // 建立 signal
    user: WritableSignal<User> = signal<User>({ name: '小明', age: 16 });

    // 直接對 signal 設定新值
    setUser() {
      this.user.set({ name: '小明', age: 24 });
    }

    // 透過 callback 對 signal 設定新值
    updateUser() {
      this.user.update(u => ({ ...u, age: u.age + 1 }));
    }

    // 透過 asReadonly 從來源取得唯讀的 signal
    readonlyUser: Signal<User> = this.user.asReadonly();

    // 來源signal變動時(且computed的signal被template使用)，會呼叫callback
    isAdult: Signal<boolean> = computed(() => this.user().age >= 18);

    // 來源signal變動時，會呼叫callback
    logEffect = effect(() => {
      console.log('User changed:', this.user());
    });
  }
  ```

### Input() 支援 required 關鍵字

可以直接在裝飾器 @Input() 加上 `required` 來強制要求父元件一定要傳入這個 input 屬性。這樣如果父元件沒有提供這個 input，開發時就會出現錯誤。

(基於裝飾器的 @InputAPI 仍可使用，不過官方在之後的版本，建議在新專案中使用基於 signal 的 input。)

```typescript
@Input({ required: true }) name1: string = '';

// Signal-based input, returns an `InputSignal<string>`.
name2 = input.required<string>();
```

### SSR 和 Hydration

v16 新出現的 Hydration（水合）功能是指在伺服器端渲染（server-side rendering, SSR）後，將靜態的 HTML 內容在瀏覽器端「補水」，讓它恢復成可互動的 Angular 應用程式。

簡單來說：

1. HTML 預渲染：伺服器先產生完整網頁 HTML，使用者打開網頁時可立即看到內容。
2. 啟用hydration：在前端自動將這些 HTML 轉化為可互動的 Angular 元件，無需重新渲染。

這項功能極大地提升了首次載入的速度與 SEO，同時保留 SPA 的互動性。  
(v16 為預覽版。 v17 開始進入穩定版，且新的專案如果使用 SSR，將預設啟用 hydration)

---

## Angular v17

### 新的區塊模板語法 (@if, @switch, @for 等)

- 推出全新的區塊模板語法(block template syntax)，帶來強大且宣告式的 API，支援控制流程、lazy loading 等功能，能有效提升開發者體驗。
- 新語法由 Angular 編譯器轉換成高效 JavaScript 指令，帶來顯著效能提升。
- 範例：

  ```html
  <!-- @if 範例 -->
  @if (isLoggedIn) {
    <p>歡迎回來！</p>
  } @else {
    <p>請先登入。</p>
  }

  <!-- @switch 範例 -->
  @switch (status) {
    @case ('loading') {
      <p>載入中...</p>
    }
    @case ('error') {
      <p>發生錯誤！</p>
    }
    @default {
      <p>完成！</p>
    }
  }

  <!-- @for 範例 -->
  @for (let item of items; track item.id) {
    <li>{{ item.name }}</li>
  } @empty {
    <li>目前沒有資料。</li>
  }
  ```

  - 這邊要特別提到新版 @for 語法強制要求使用 track，並採用全新 diff 演算法，確保更高的 diffing 效能。
  - @for 的 track 只需寫成一個表達式，不像以往必須在元件 class 內額外定義方法，語法更簡單易用。
  - @for 支援 @empty 區塊，能針對集合為空時快速呈現替代內容。

- 舊版專案可透過以下 CLI 指令轉換成新版的語法

  ```bash
  ng generate @angular/core:control-flow
  ```

### 新的延遲載入語法: `@defer`

- Deferrable Views（可延遲載入的視圖） 是 Angular 17 新推出的懶載入機制，利用 block syntax 實現更強大且簡潔的延遲載入功能。
- 開發者只需一行宣告式語法`@defer`，即可延遲載入元件及其所有依賴，不再需要手動管理 ViewContainerRef、清理、錯誤處理等繁瑣流程。
- Angular 編譯器會自動將 defer block 內的元件、指令、pipes 轉為動態 import，並管理載入狀態切換。
- 支援多種觸發條件，包括：
  - `on idle`：瀏覽器空閒時延遲載入 (@defer預設行為)
  - `on immediate`：立刻開始延遲載入
  - `on timer`： 使用計時器延遲載入
  - `on viewport`：元件進入視窗時延遲載入，也可另外指定參考元素
  - `on interaction`、`on hover`：用戶互動或滑鼠懸停時延遲載入
  - `when <expr>`：自訂條件
- 可用 `@placeholder`、`@loading`、`@error` 區塊管理載入前、載入中及失敗的 UI 狀態。
- 可用 `prefetch` 提前預取依賴。
- 範例:

  ```html
  <!-- 最基本的延遲載入區塊 -->
  @defer {
    <comment-list />
  }

  <!-- 當區塊進入視窗才開始載入 -->
  @defer (on viewport) {
    <comment-list />
  }

  <!-- 當區塊進入視窗才開始載入，並會在瀏覽器空閒時提前預取依賴 -->
  @defer (on viewport; prefetch on idle) {
    <comment-list />
  }

  <!-- 當參考元素進入視窗才開始載入 -->
  @defer (on viewport(commentsAnchor)) {
    <comment-list />
  }

  <!-- 延遲一段時間後再開始載入(毫秒)，並包含 placeholder、載入中與錯誤的狀態處理 -->
  @defer (on timer(2000)) {
    <heavy-chart />
  } @placeholder {
    <div class="skeleton-chart">圖表載入準備中...</div>
  } @loading {
    正在載入圖表…
  } @error {
    圖表載入失敗，請重試。
  }

  <!-- 當 isDetailsReady 為 truthy 時才載入 -->
  @defer (when isDetailsReady) {
    <details-view [details]="details" />
  }
  ```

- @defer 注意事項：
  - when 從 falsy -> truthy 轉變時觸發一次載入。要注意載入完成後不會自動移除，也不會因為條件再變回 falsy 而回復成 placeholder。
  - 可同時設定多個觸發條件(例如`(when condition; on timer(5000); prefetch on idle)`)，且任一先達成即開始載入。
  - prefetch 不會執行初始化，只抓資源；正式觸發仍需達成觸發條件。

### v17 的其他更新

- Signals 等 API 開始陸續進入穩定版本
- 新專案預設啟用 Standalone API，CLI 新增 `ng new --standalone`。
- 新專案使用 SSR，預設啟用 hydration。

---

## 小結

本篇文章中快速帶過 Angular v15 到 v17 我覺得比較重大的更新，像是 `standalone` 元件、`signals`、新的區塊模板語法(`@if, @switch, @for`)，以及更強大的 `@defer` 延遲載入機制。使用這些新東西漸漸替換掉之前的舊有寫法，應能讓我們的開發流程更簡潔、運作效能更好。

下一篇會繼續介紹 v18 之後的重點更新，敬請期待！

### 參考來源

- [Angular 官方文件](https://angular.dev/)
- [Angular Blog](https://blog.angular.io/)
- [Angular 更新日誌](https://github.com/angular/angular/blob/main/CHANGELOG.md)

---
