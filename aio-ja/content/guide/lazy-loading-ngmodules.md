# フィーチャーモジュールの遅延ロード

デフォルトでは、NgModuleは即時にロードされます。つまり、アプリケーションがロードされるとすぐに、すぐに必要であるかどうかにかかわらず、すべてのNgModuleがロードされます。多数のルートをもつ大規模なアプリケーションの場合は、遅延ロード（必要に応じてNgModuleをロードする設計パターン）を検討してください。
遅延ロードは、初期バンドルサイズを小さく保つのに役立ち、これによりロード時間が短縮されます。

<div class="alert is-helpful">

このページで説明する2つの遅延ロードされたモジュールをもつ最終的なサンプルについては
<live-example></live-example>を参照してください。

</div>

{@a lazy-loading}

## 遅延ロードの基本 {@a lazy-loading-basics}

このセクションでは、遅延ロードされるルートを設定するための基本的な手順を紹介します。
ステップバイステップの例については、このページの[ステップバイステップのセットアップ](#step-by-step)セクションを参照してください。

Angularモジュールを遅延ロードするには、次のように、`AppRoutingModule`の`routes`設定で`loadChildren`を(`component`の代わりに)使用します。

<code-example header="AppRoutingModule (excerpt)">

const routes: Routes = [
  {
    path: 'items',
    loadChildren: () => import('./items/items.module').then(m => m.ItemsModule)
  }
];

</code-example>

遅延ロードされるモジュールのルーティングモジュールで、そのコンポーネントのためのルートを追加します。

<code-example header="Routing module for lazy loaded module (excerpt)">

const routes: Routes = [
  {
    path: '',
    component: ItemsComponent
  }
];

</code-example>

また、`AppModule`から`ItemsModule`を必ず削除します。
遅延ロードのモジュールのステップバイステップの説明については、このページの次のセクションに進みましょう。

<a id="step-by-step"></a>

## ステップバイステップのセットアップ {@a step-by-step-setup}

遅延ロードするフィーチャーモジュールをセットアップするための主なステップが2つあります:

1. Angular CLIで `--route` フラグを使ってフィーチャーモジュールを作成する
1. ルート(route)を設定する

### アプリケーションをセットアップする

まだアプリケーションを作成していない場合、次のステップにしたがってCLIを使用してアプリケーションを作成してください。
すでに作成している場合は[ルート(route)を設定する](#config-routes)に進んでください。

<!-- vale Angular.Google_WordListWarnings = NO -->

次のコマンドを実行してください。ここでの`customer-app`はアプリケーション名です。

<!-- vale Angular.Google_WordListWarnings = YES -->

<code-example format="shell" language="shell">

ng new customer-app --routing

</code-example>

これにより、`customer-app`というアプリケーションが作成され、さらに`--routing`フラグによって
`app-routing.module.ts`というファイルが生成されます。
このファイルはフィーチャーモジュールの遅延ロードをセットアップするために必要なファイルの1つです。
コマンド`cd customer-app`を実行してプロジェクトに移動してください。

<div class="alert is-helpful">

`--routing`オプションはAngular CLI 8.1以上で有効です。
[プロジェクトの更新](guide/updating)を参照してください。

</div>

### ルーティングをもつフィーチャーモジュールを作成する

次に、ルーティングするためのコンポーネントとフィーチャーモジュールが必要になります。
作成するために、ターミナルで次のコマンドを実行してください。`customers`はフィーチャーモジュールの名前です。`customers`フィーチャーモジュールをロードするパスもまた`--route`オプションを使用して`customers`に指定します。

<code-example format="shell" language="shell">

ng generate module customers --route customers --module app.module

</code-example>

これにより、`customers.module.ts`ファイルで定義された新しい遅延ロード可能なフィーチャーモジュール`CustomersModule`と、`customers-routing.module.ts`ファイルで定義されたルーティングモジュール`CustomersRoutingModule`をもつ、`customers`ディレクトリが作成されます。このコマンドは自動的に`CustomersComponent`を宣言し、新しいフィーチャーモジュール内に`CustomersRoutingModule`をインポートします。

新しいモジュールは遅延ロードされることを意図しているため、コマンドは新しいフィーチャーモジュールへの参照をルートアプリケーションモジュールの`app.module.ts`ファイルに**追加しません**。
代わりに、`--module`オプションで指定したモジュール内の`routes`配列に、宣言したルート(route)`customers`を追加します。

<code-example header="src/app/app-routing.module.ts" path="lazy-loading-ngmodules/src/app/app-routing.module.ts" region="routes-customers"></code-example>

遅延ロード構文では、関数が後ろに続く`loadChildren`を使うことに注目してください。関数はブラウザ組み込みの動的インポート用の`import('...')`構文を用います。
インポートのパスはそのモジュールへの相対パスです。

<div class="callout is-helpful">

<header>文字列ベースの遅延ロード</header>

Angularバージョン8では、`loadChildren`のルート指定のための文字列構文は、`import()`構文の推奨によって[非推奨に](guide/deprecations#loadchildren-string-syntax)なりました。
`tsconfig`ファイルに遅延ロードされるルートを含めることで、文字列ベースの遅延ロード(`loadChildren: './path/to/module#Module'`)の使用を選択できます。それらはコンパイルで遅延ロードされるファイルを含みます。

デフォルトでAngular CLIは、`import()`構文で使用することを意図したより厳密なファイル構成のプロジェクトを生成します。

</div>

#### もう1つのフィーチャーモジュールを追加する

同じコマンドを使って、2つ目の遅延ロードのフィーチャーモジュールとルーティングとそのスタブコンポーネントを作成します。

<code-example format="shell" language="shell">

ng generate module orders --route orders --module app.module

</code-example>

これにより、`OrdersModule`と`OrdersRoutingModule`と、新しい`OrdersComponent`ソースファイルが含まれる、`orders`という新しいディレクトリが作成されます。
`--routes`オプションを使用して指定した`orders`ルート(route)は、遅延ロード構文を使用して`app-routing.module.ts`ファイル内の`routes`配列に追加されます。

<code-example header="src/app/app-routing.module.ts" path="lazy-loading-ngmodules/src/app/app-routing.module.ts" region="routes-customers-orders"></code-example>

### UIをセットアップする

アドレスバーにURLを入力することもできますが、ナビゲーションUIのほうがユーザーにとって簡単でより一般的です。
`app.component.html`のデフォルトのプレースホルダーマークアップをカスタムナビゲーションに置き換えて、
ブラウザ内でモジュールに簡単にナビゲートすることができるようにしてみましょう:

<code-example path="lazy-loading-ngmodules/src/app/app.component.html" header="app.component.html" region="app-component-template" header="src/app/app.component.html"></code-example>

今まであなたが作成したアプリケーションをブラウザで確認するために、ターミナルで次のコマンドを入力してください:

<code-example format="shell" language="shell">

ng serve

</code-example>

<!-- vale Angular.Google_WordListWarnings = NO -->

そのあとに`localhost:4200`にアクセスしてみましょう。“customer-app” という文字列と3つのボタンが表示されているはずです。

<!-- vale Angular.Google_WordListWarnings = YES -->

<div class="lightbox">

<img alt="three buttons in the browser" src="generated/images/guide/lazy-loading-ngmodules/three-buttons.png" width="300">

</div>

これらのボタンは機能します。なぜならAngular CLIが、フィーチャーモジュールへのルート(route)を`app-routing.module.ts`内の`routes`配列に自動的に追加したからです。

<a id="config-routes"></a>

### インポートとルート(route)設定

Angular CLIは各フィーチャーモジュールをアプリケーションレベルでのルート(route)マップへ自動的に追加します。
デフォルトのルート(route)を追加してこれを完成させましょう。`app-routing.module.ts`ファイル内の`routes`配列を次のように更新してください:

<code-example header="src/app/app-routing.module.ts" path="lazy-loading-ngmodules/src/app/app-routing.module.ts" id="app-routing.module.ts" region="const-routes"></code-example>

最初の2つのパスは`CustomersModule`と`OrdersModule`へのルート(route)です。
最後のエントリーはデフォルトのルート(route)です。空のパスは、それまでのパスにマッチしないものすべてにマッチします。


### フィーチャーモジュールの内部

次に、`customers.module.ts`ファイルを見てください。
もしあなたがAngular CLIを使用していて、このページに記載されている手順にしたがっている場合は、ここで何もする必要はありません。

<code-example header="src/app/customers/customers.module.ts" path="lazy-loading-ngmodules/src/app/customers/customers.module.ts" id="customers.module.ts" region="customers-module"></code-example>

`customers.module.ts`ファイルでは、`customers-routing.module.ts`と`customers.component.ts`をインポートします。
`CustomersModule`が自分自身のルーティングモジュールにアクセスするために`@NgModule`の`imports`配列に`CustomersRoutingModule`を追加します。
そして、`CustomersComponent`は`declarations`配列に配置されます。これは`CustomersComponent`が`CustomersModule`に属することを意味します。

次に、`app-routing.module.ts`はフィーチャーモジュール`customers.module.ts`をJavaScriptの動的インポートを使用してインポートします。

フィーチャー固有のルート定義ファイル`customers-routing.module.ts`はJavaScriptのインポート文を使用して、`customers.component.ts`内で定義された自身のフィーチャーコンポーネントをインポートします。
それから`CustomersComponent`に空のパスをマップします。

<code-example header="src/app/customers/customers-routing.module.ts" path="lazy-loading-ngmodules/src/app/customers/customers-routing.module.ts" id="customers-routing.module.ts" region="customers-routing-module"></code-example>

`AppRoutingModule`内のパスがすでに`customers`に設定されているため、ここの`path`には空文字列が設定されています。つまり、この`CustomersRoutingModule`内のルート(route)はすでに`customers`コンテキスト内にあります。
このルーティングモジュール内のすべてのルート(route)は、子ルートとなります。

他のフィーチャーモジュールがもつルーティングモジュールも同様に設定されます。

<code-example header="src/app/orders/orders-routing.module.ts (excerpt)" path="lazy-loading-ngmodules/src/app/orders/orders-routing.module.ts" id="orders-routing.module.ts" region="orders-routing-module-detail"></code-example>

### 動作の確認をする

モジュールが実際に遅延ロードされていることをChromeの開発者ツールで確認することができます。
In Chrome, open the developer tools by pressing `Cmd+Option+i` on a Mac or `Ctrl+Shift+j` on a PC and go to the Network Tab.

<div class="lightbox">

<img alt="lazy loaded modules diagram" src="generated/images/guide/lazy-loading-ngmodules/network-tab.png" width="600">

</div>

Click on the Orders or Customers button.
If you see a chunk appear, everything is wired up properly and the feature module is being lazy loaded.
A chunk should appear for Orders and for Customers but only appears once for each.

<div class="lightbox">

<img alt="lazy loaded modules diagram" src="generated/images/guide/lazy-loading-ngmodules/chunk-arrow.png" width="600">

</div>

To see it again, or to test after making changes, click the circle with a line through it in the upper left of the Network Tab:

<div class="lightbox">

<img alt="lazy loaded modules diagram" src="generated/images/guide/lazy-loading-ngmodules/clear.gif" width="200">

</div>

そのあとでプラットフォームに応じて、`Cmd+r`または`Ctrl+r`でリロードしてください。

## `forRoot()`と`forChild()`

あなたはAngular CLIが`RouterModule.forRoot(routes)`を `AppRoutingModule` の`imports`配列に追加したことに気づいたかもしれません。
これにより、Angularは`AppRoutingModule`がルーティングモジュールであることと、`forRoot()`によってルート(root)のルーティングモジュールであることを認識します。
これは渡したすべてのルート(route)を設定して、ルーターディレクティブへアクセスできるようにし、`Router`サービスを登録します。
`forRoot()`は`AppRoutingModule`の中で、アプリケーション内で1回だけ使用してください。

Angular CLIは、フィーチャールーティングモジュールにも`RouterModule.forChild(routes)`を追加します。
これにより、Angularはそのルート(route)リストが追加のルート(route)の提供のみに責任をもつことと、フィーチャーモジュールを対象としていることを認識します。
`forChild()`は複数のモジュールで使用することができます。

`forRoot()`メソッドはRouterのための*グローバルな*インジェクター設定を管理します。
`forChild()`メソッドはインジェクター設定を持ちません。
それは`RouterOutlet`や`RouterLink`のようなディレクティブを使います。
詳しくは、 [シングルトンサービス](guide/singleton-services) ガイドの中の [`forRoot()` パターン](guide/singleton-services#forRoot) セクションを参照してください。

<a id="preloading"></a>

## プリロード

プリロードは、アプリケーションの一部をバックグラウンドでロードすることでUXを向上させます。
モジュールまたはコンポーネントデータをプリロードできます。

### モジュールをプリロードする {@a preloading-modules}

モジュールのプリロードはアプリケーションの一部をバックグラウンドでロードすることでUXを向上させるため、ユーザーはルートをアクティブ化するときにその要素がダウンロードされるのを待つ必要がありません。

遅延ロードされたモジュールすべてのプリロードを有効にするには、Angularの`router`から`PreloadAllModules`トークンをインポートします。

<code-example header="AppRoutingModule (excerpt)">

import { PreloadAllModules } from '&commat;angular/router';

</code-example>

引き続き`AppRoutingModule`で、`forRoot()`でプリロード戦略を指定します。

<code-example header="AppRoutingModule (excerpt)">

RouterModule.forRoot(
  appRoutes,
  {
    preloadingStrategy: PreloadAllModules
  }
)

</code-example>

### コンポーネントデータをプリロードする {@a preloading-component-data}

コンポーネントデータをプリロードするために、`resolver`を使用できます。
リゾルバは、ページを完全に表示するために必要なすべてのデータが利用可能になるまで、ページのロードをブロックしてUXを向上させます。

#### リゾルバ {@a resolvers}

リゾルバサービスを作成します。
Angular CLIでサービスを生成するコマンドは次のとおりです:

<code-example format="shell" language="shell">

ng generate service &lt;service-name&gt;

</code-example>

In the newly created service, implement the `Resolve` interface provided by the `&commat;angular/router` package:

<code-example header="Resolver service (excerpt)">

import { Resolve } from '&commat;angular/router';

&hellip;

/* An interface that represents your data model */
export interface Crisis {
  id: number;
  name: string;
}

export class CrisisDetailResolverService implements Resolve&lt;Crisis&gt; {
  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable&lt;Crisis&gt; {
    // your logic goes here
  }
}

</code-example>

このリゾルバをモジュールのルーティングモジュールにインポートします。

<code-example header="Feature module's routing module (excerpt)">

import { CrisisDetailResolverService } from './crisis-detail-resolver.service';

</code-example>

コンポーネントの`route`設定に`resolve`オブジェクトを追加します。

<code-example header="Feature module's routing module (excerpt)">

{
  path: '/your-path',
  component: YourComponent,
  resolve: {
    crisis: CrisisDetailResolverService
  }
}

</code-example>


In the component's constructor, inject an instance of the `ActivatedRoute` class that represents the current route.

<code-example header="Component's constructor (excerpt)">

import { ActivatedRoute } from '&commat;angular/router';

&commat;Component({ &hellip; })
class YourComponent {
  constructor(private route: ActivatedRoute) {}
}

</code-example>

Use the injected instance of the `ActivatedRoute` class to access `data` associated with a given route.

<code-example header="Component's ngOnInit lifecycle hook (excerpt)">

import { ActivatedRoute } from '&commat;angular/router';

&commat;Component({ &hellip; })
class YourComponent {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.data
      .subscribe(data =&gt; {
        const crisis: Crisis = data.crisis;
        // &hellip;
      });
  }
}

</code-example>

動作例と詳細については、[ルーティングチュートリアルのプリロードのセクション](guide/router-tutorial-toh#preloading-background-loading-of-feature-areas)を参照してください。

## Troubleshooting lazy-loading modules

A common error when lazy-loading modules is importing common modules in multiple places within an application.
Test for this condition by first generating the module using the Angular CLI and including the `--route route-name` parameter, where `route-name` is the name of your module.
Next, create the module without the `--route` parameter.
If `ng generate module` with the `--route` parameter returns an error, but runs correctly without it, you might have imported the same module in multiple places.

Remember, many common Angular modules should be imported at the base of your application.

For more information on Angular Modules, see [NgModules](guide/ngmodules).

## NgModuleとルーティングの詳細

あなたはこちらにも興味があるかもしれません:

* [ルーティングとナビゲーション](guide/router)
* [プロバイダー](guide/providers)
* [フィーチャーモジュールの種類](guide/module-types)
* [Route-level code-splitting in Angular](https://web.dev/route-level-code-splitting-in-angular/)
* [Route preloading strategies in Angular](https://web.dev/route-preloading-in-angular/)

<!-- links -->

<!-- external links -->

<!-- end links -->

@reviewed 2022-02-28
