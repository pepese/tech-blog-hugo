---
title: "Cognitoを使ってユーザ認証して画面遷移するAngularアプリ"
date: 2017-05-29T07:49:45+09:00
slug: "angular-router-cognito"
tags:
- angular
- javascript
- typescript
- aws
- cognito
draft: false
archives:
- 2017/05
---

以下の記事で紹介したCognitoで認証するAngularアプリにAngular Routerで画面遷移ロジックを加えたアプリを作成する。

- [Cognitoを使ってAngularアプリからユーザ認証する](https://blog.pepese.com/entry/angular-cognito/)

<!--more-->

`@angular/router` を使うことでAngularで画面遷移を実現できる。

# アプリケーションの仕様

- `/login` と `/menu` がある
- `/login` 画面では、Amazon Cognito User Pool を使ってSignUp、SignInが可能
- `/login` 画面でSignInすると `/menu` 画面へ遷移する
- `/menu` 画面では、Session StorageにSignInユーザ情報が無い場合、 `/login` 画面へ遷移する

# アプリケーションの作成手順

以下のコマンドでプロジェクトおよび必要なファイルを作成する。

```sh
$ ng new cognito-js --style=scss
$ cd cognito-js
$ npm install amazon-cognito-identity-js --save
$ ng generate class app.routing
$ ng generate service services/cognito
$ ng generate component components/login
$ ng generate component components/menu
```

以下の順でソースを編集する。

- src/tsconfig.app.json
- src/app/app.routing.ts
    - 遷移する画面（Component）情報をここへ集約する
- src/app/app.module.ts
- src/environments/environment.ts
- src/app/app.component.ts
- src/app/services/cognito.service.ts
- src/app/components/login/login.component.ts
    - `/login` 画面のComponent
- src/app/components/menu/menu.component.ts
    - `/menu` 画面のComponent
- src/app/components/login/login.component.html
    - `/login` 画面
- src/app/components/menu/menu.component.html
    - `/menu` 画面
- src/app/app.component.html

## src/tsconfig.app.json

```javascript
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "module": "es2015",
    "baseUrl": "",
    "types": ["node"]  // ここを追加
  },
  "exclude": [
    "test.ts",
    "**/*.spec.ts"
  ]
}
```

## src/app/app.routing.ts

```typescript
import { ModuleWithProviders }  from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { LoginComponent } from './components/login/login.component';
import { MenuComponent } from './components/menu/menu.component';

const appRoutes: Routes = [
    {
        path: 'login',
        component: LoginComponent
    },
    {
        path: 'menu',
        component: MenuComponent
    },
    {
        path: '',
        redirectTo: '/login',
        pathMatch: 'full'
    }
];

export const routing: ModuleWithProviders = RouterModule.forRoot(appRoutes);
```

`{ path: '', redirectTo: '/login', pathMatch: 'full' }` の記述でルートパスへアクセスされると、 `/login` 画面へリダイレクトされる。

## src/app/app.module.ts

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { LoginComponent } from './components/login/login.component';
import { MenuComponent } from './components/menu/menu.component';
import { routing } from './app.routing'; // 追記

@NgModule({
  declarations: [
    AppComponent,
    LoginComponent,
    MenuComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    routing // 追記
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

## src/environments/environment.ts

```typescript
export const environment = {
  production: false,
  region: 'ap-northeast-1',
  userPoolId: 'ap-northeast-1_xxxxxxxxx',
  clientId: 'xxxxxxxxxxxxxxxxxxxxxxxxxx'
};
```

## src/app/app.component.ts

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  title = 'Cognito Sample'; // ここを好きなタイトルに編集
}
```

## src/app/services/cognito.service.ts

```typescript
import { Injectable } from '@angular/core';
import * as AWS from "aws-sdk";
import { CognitoUserPool, CognitoUserAttribute, CognitoUser, AuthenticationDetails } from 'amazon-cognito-identity-js';
import { environment } from '../../environments/environment';

@Injectable()
export class CognitoService {
  userPool = null;

  constructor() {
    AWS.config.region = environment.region;
    const data = { UserPoolId: environment.userPoolId, ClientId: environment.clientId};
    this.userPool = new CognitoUserPool(data);
  }

  signUp(username, password, email, phone) {
    const userData = {
      Username : username,
      Pool : this.userPool,
      Storage: sessionStorage
    };
    let attributeList = [];
    const dataEmail = {
      Name : 'email',
      Value : email
    };
    const dataPhoneNumber = {
      Name : 'phone_number',
      Value : phone
    };
    let attributeEmail = new CognitoUserAttribute(dataEmail);
    let attributePhoneNumber = new CognitoUserAttribute(dataPhoneNumber);
    attributeList.push(attributeEmail);
    attributeList.push(attributePhoneNumber);
    this.userPool.signUp(username, password, attributeList, null, function(err, result){
      if (err) {
        alert(err);
        return;
      }
      const cognitoUser = result.user;
      alert("SignUp is success!\nUser name is " + cognitoUser.getUsername() + ".\nYou need to check your SMS or E-Mail.");
    });
    return;
  }

  confirmRegistration(username, verification_code) {
    const userData = {
      Username : username,
      Pool : this.userPool,
      Storage: sessionStorage
    };
    const cognitoUser = new CognitoUser(userData);
    cognitoUser.confirmRegistration(verification_code, true, function(err, result) {
      if (err) {
        alert(err);
        return;
      }
      alert('Registration is success!');
      console.log('call result: ' + result);
    });
    return;
  }

  signIn(username, password, callback) {
    const userData = {
      Username : username,
      Pool : this.userPool,
      Storage: sessionStorage
    };
    const cognitoUser = new CognitoUser(userData);
    const authenticationData = {
        Username : username,
        Password : password
    };
    const authenticationDetails = new AuthenticationDetails(authenticationData);
    cognitoUser.authenticateUser(authenticationDetails, {
      onSuccess: function (result) {
        alert('SignIn is success!');
        console.log('access token + ' + result.getAccessToken().getJwtToken());
        if(callback) callback();
      },
      onFailure: function(err) {
        alert(err);
        // UserNotConfirmedException: User is not confirmed.
      }
    });
    return;
  }

  getSignInUserNmae() {
    let username_key = 'CognitoIdentityServiceProvider.' + environment.clientId + '.LastAuthUser';
    return sessionStorage.getItem(username_key);
  }

  signOut(callback) {
    sessionStorage.clear();
    if(callback) callback();
  }
}
```

## src/app/components/login/login.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { CognitoService } from '../../services/cognito.service';
import { Router } from '@angular/router';
import { routing } from '../../app.routing';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.scss'],
  providers: [CognitoService]
})
export class LoginComponent implements OnInit {
  username = '';
  password = ``;
  email = ``;
  phone = ``;
  verification_code = ``;

  constructor(
    private cognitoService: CognitoService,
    private router: Router
  ) { }

  ngOnInit() {
  }

  siginUp() {
    this.cognitoService.signUp(this.username, this.password, this.email, this.phone);
  }

  confirmRegistration() {
    this.cognitoService.confirmRegistration(this.username, this.verification_code);
  }

  signIn() {
    // Session Storageにセッション情報が格納されるまで待ってから画面遷移
    let gotoMenu = function (router: Router, cognitoService: CognitoService) {
      let timerID = setInterval(function(){
        if(cognitoService.getSignInUserNmae()){
          //wait終了時の後処理
          router.navigate(['/menu']);
          clearInterval(timerID);
          timerID = null;
        }
      }, 100);
    }
    this.cognitoService.signIn(this.username, this.password, gotoMenu(this.router, this.cognitoService));
  }
}
```

`signIn()` メソッドでは `cognitoUser.authenticateUser` の `callback.onSuccess` の段階で Session Storage にセッション情報が格納されていなかったので、 `setInterval` でセッションが格納されるまで wait するロジックを追加してみた。

## src/app/components/menu/menu.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { CognitoService } from '../../services/cognito.service';
import { Http, Response, Headers, RequestOptions } from '@angular/http';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/map';
import { environment } from '../../../environments/environment';
import { Router } from '@angular/router';
import { routing } from '../../app.routing';

@Component({
  selector: 'app-menu',
  templateUrl: './menu.component.html',
  styleUrls: ['./menu.component.scss'],
  providers: [CognitoService]
})
export class MenuComponent implements OnInit {
  url = '';
  result = '';
  gotoLogin = function (router: Router) {
    router.navigate(['/login']);
  }

  constructor(
    private http: Http,
    private cognitoService: CognitoService,
    private router: Router
  ) { }

  ngOnInit() {
    if(!this.cognitoService.getSignInUserNmae()) {
      this.cognitoService.signOut(this.gotoLogin(this.router));
    }
  }

  deleteSession() {
    this.cognitoService.signOut(this.gotoLogin(this.router));
  }

  get() {
    let username_key = 'CognitoIdentityServiceProvider.' + environment.clientId + '.LastAuthUser';
    let username = sessionStorage.getItem(username_key);
    let idToken_key = 'CognitoIdentityServiceProvider.' + environment.clientId + '.' + username + '.idToken';
    let idToken = sessionStorage.getItem(idToken_key);
    let headers = new Headers({ 'Authorization': idToken });
    let options = new RequestOptions({ headers: headers });
    return this.http.get(this.url, options)
                    .map(res => res.text())
                    .subscribe(t => this.result = t);
  }

}
```

## src/app/components/login/login.component.html

```html
<div>
  <h2>サインアップ</h2>
  <p>以下の項目を入力してユーザを新規に仮登録します。</p>
  <table>
    <tr><td>ユーザ名：</td><td><input type="text" [(ngModel)]="username"></td><tr>
    <tr><td>パスワード：</td><td><input type="text" [(ngModel)]="password"></td></tr>
    <tr><td>メールアドレス：</td><td><input type="text" [(ngModel)]="email"></td></tr>
    <tr><td>携帯番号：</td><td><input type="text" [(ngModel)]="phone"></td></tr>
  </table>
  <button (click)="siginUp()">サインアップ</button>
</div>
<hr />
<div>
  <h2>認証コードの確認</h2>
  <p>サインアップしたユーザにメールやSMSで送付された認証コードを確認し、ユーザを本登録します。</p>
  <p>認証コードは6桁数字です。</p>
  <table>
    <tr><td>ユーザ名：</td><td>{{username}}</td></tr>
    <tr><td>認証コード：</td><td><input type="text" [(ngModel)]="verification_code"></td></tr>
  </table>
  <button (click)="confirmRegistration()">確認</button>
</div>
<hr />
<div>
  <h2>サインイン</h2>
  <p>本登録したユーザでサインイン（ログイン）します。</p>
  <table>
    <tr><td>ユーザ名：</td><td>{{username}}</td></tr>
    <tr><td>パスワード：</td><td>{{password}}</td></tr>
  </table>
  <button (click)="signIn()">サインイン</button>
</div>
```

## src/app/components/menu/menu.component.html

```html
<div>
  <a routerLink="/login" (click)="deleteSession()">ログアウト</a>
</div>
<div>
  <h2>GET アクセス</h2>
  <p>Session StorageのidTokenをAuthorizationヘッダへ付与してGetリクエストを送付します。</p>
  <p>API GatewayのAuthorizationなどで試してみてください。</p>
  <table>
    <tr><td>URL：　</td><td><input type="text" [(ngModel)]="url"></td></tr>
  </table>
  <button (click)="get()">Send</button>
  <table>
    <tr><td>{{result}}</td></tr>
  </table>
</div>
```

## src/app/app.component.html

```html
<h1>
  {{title}}
</h1>
<hr />
<router-outlet></router-outlet>
```

`<router-outlet>` の箇所が `Router` によって画面が切り替えられる。


# 参考

- [Angular 2 画面遷移](http://thread.main.jp/javascript/angularjs23.html)
- [[ng-japan/hands-on] Chapter 6: ルーティング](https://github.com/ng-japan/hands-on/tree/master/courses/tutorial/ch-6)
- [公式:ROUTING](https://angular.io/docs/ts/latest/tutorial/toh-pt5.html)
- [公式:ROUTING & NAVIGATION](https://angular.io/docs/ts/latest/guide/router.html)
