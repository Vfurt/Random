[![N|Solid](https://sentry-brand.storage.googleapis.com/sentry-logo-black.png)](https://sentry.io)

Sentry é uma ferramenta Open Source para captação de eventos e exceções em tempo real, capaz de fornecer dados capazes de facilitar a monitoração, reprodução e resolução de problemas inesperados da aplicação.

Enquanto New Relic oferece recursos para facilitar a análise de performance dos aplicativos, Sentry oferece recursos como:

#### Relatórios de erros
 - Updates de erros e notificações da aplicação em tempo real
![N](https://i.imgur.com/RI6rP65.png)
 - Informa o contexto da aplicação durante o erro
![N](https://i.imgur.com/FmlXTgW.png)
![N](https://i.imgur.com/RA4A5ru.png)
 - Notificações por E-mail, SMS e Slack
 - Ferramente integrada à aplicação para feedback do usuário  customizável, anexado ao erro
 ![N|Solid](https://docs.sentry.io/assets/user_feedback_widget-d73dfdf1999502c73317f8155a05e48e163e8772b4bfc771485086ea043652de.png)
 - Controle de erros por releases e ambientes de desenvolvimento
 ![N|Solid](https://docs.sentry.io/assets/environments/env_dropdown-955fde42ed5aff1eae3320be20d4cad32fe207a3f0eaed31c61d77cb23e9a731.png)
 - Métricas da aplicação (Plano Team/Business)

#### Workflow
 - Integração dos erros e eventos ao workflow do Jira
 - Integração com repositórios do Bitbucket, Github, Gitlab, Azure DevOps para monitoração de releases
 - Suporte para diversas plataformas: Javascript, .Net, Node, Java, Android, iOS 
 
## Planos e licença
Sentry.io conta com as licenças MIT e Apache, é totalmente open source e pode ser utilizado por empresas por completo desde que não seja comercializado (BSL), além de oferecer os serviços cloud Team e business.

#### Team: $26/mês
- Até 100.000 eventos
- Integração automática com Jira e repositórios
- Notificações semanais
- Discover (ferramenta de métricas da aplicação)

#### Business: $89/mês
- Single Sign-On
- Filtros customizáveis para descarte de eventos indesejáveis
- Limitar Eventos por aplicações
- Permite envio de eventos para aplicações de terceiros, como Amazon SQS
- Exportar dados de eventos por REST API
- Métricas customizáveis na ferramenta Discover


## Configuração
### Vue.JS: https://docs.sentry.io/clients/javascript/integrations/#vue
1. Instalar @sentry/browser:
```
# Yarn
yarn add @sentry/browser
yarn add @sentry/integrations

# npm
npm install @sentry/browser
npm install @sentry/integrations
```

2. Inicializar nova instância Sentry e configurar o ErrorHandler
```
import Vue from 'vue'
import * as Sentry from '@sentry/browser';
import { Vue as VueIntegration } from '@sentry/integrations';

Sentry.init({
  dsn: 'https://SENTRY_USER_ID.ingest.sentry.io/5266297',
  integrations: [new VueIntegration({Vue, attachProps: true})],
});
```

### Angular: https://docs.sentry.io/clients/javascript/integrations/#angular
1. Instalar @sentry/browser:
```
# Yarn
yarn add @sentry/browser

# npm
npm install @sentry/browser
```

2. Inicializar nova instância Sentry e configurar o ErrorHandler
```
import { BrowserModule } from "@angular/platform-browser";
import { NgModule, ErrorHandler, Injectable } from "@angular/core";
import { HttpErrorResponse } from "@angular/common/http";

import { environment } from "../environments/environment";
import { AppComponent } from "./app.component";

import * as Sentry from "@sentry/browser";

Sentry.init({
  dsn: "https://SENTRY_USER_ID.ingest.sentry.io/5266297",
  // TryCatch precisa ser configurado para desabilitar XMLHttpRequest para que possamos controlar as exceções no módulo http manualmente
  integrations: [new Sentry.Integrations.TryCatch({
    XMLHttpRequest: false,
  })],
});

@Injectable()
export class SentryErrorHandler implements ErrorHandler {
  constructor() {}

  extractError(error) {
    if (error && error.ngOriginalError) {
      error = error.ngOriginalError;
    }

    // Configura mensagens e objeto de erro
    if (typeof error === "string" || error instanceof Error) {
      return error;
    }

    // Se erro for do módulo http, tenta extrair informações
    if (error instanceof HttpErrorResponse) {
      // O erro de exceção do módulo http pode ser um objeto `Error` no qual podemos usar diretamente
      if (error.error instanceof Error) {
        return error.error;
      }

      // ... ou um `ErrorEvent`, que pode nos informar a mensagem de erro
      if (error.error instanceof ErrorEvent) {
        return error.error.message;
      }

      // ... ou o próprio body, para informar a mensagem de erro
      if (typeof error.error === "string") {
        return `Server returned code ${error.status} with body "${error.error}"`;
      }

      // Se não temos informações detalhadas, enviamos a mensagem da própria exceção
      return error.message;
    }

    return null;
  }

  handleError(error) {
    let extractedError = this.extractError(error) || "Handled unknown error";

    // Captura exceção e a envia para Sentry
    const eventId = Sentry.captureException(extractedError);
    
    // Mostra o erro no console durante desenvolvimento
    if (!environment.production) {
      console.error(extractedError);
    }
    
    // Opcional: mostra dialogo de feedback do usuário para fornecer detalhes do erro
    Sentry.showReportDialog({ eventId });
  }
}

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [{ provide: ErrorHandler, useClass: SentryErrorHandler }],
  bootstrap: [AppComponent]
})

export class AppModule {}
```

### AngularJS: https://docs.sentry.io/clients/javascript/integrations/#angularjs
1. Instalar @sentry/browser:
```
# Yarn
yarn add @sentry/browser @sentry/integrations

# npm
npm install @sentry/browser @sentry/integrations
```

2. Inicializar nova instância Sentry
```
import angular from 'angular';
import * as Sentry from '@sentry/browser';
import { Angular as AngularIntegration } from '@sentry/integrations';

// Após importar AngularJS 
Sentry.init({
  dsn: 'https://SENTRY_USER_ID.ingest.sentry.io/5266297',
  integrations: [
    new AngularIntegration(),
  ],
});

// aplica ngSentry como uma dependencia
angular.module('yourApplicationModule', ['ngSentry']);
```
