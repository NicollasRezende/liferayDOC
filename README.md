# Guia Completo de Desenvolvimento Front-End para Liferay DXP

## Sumário

1. [Introdução ao Desenvolvimento Front-End no Liferay](#1-introdução-ao-desenvolvimento-front-end-no-liferay)
2. [Branding e Temas](#2-branding-e-temas)
3. [Fragmentos de Página](#3-fragmentos-de-página)
4. [Templates de Widgets e Apresentação de Conteúdo](#4-templates-de-widgets-e-apresentação-de-conteúdo)
5. [Layouts e Templates de Layout](#5-layouts-e-templates-de-layout)
6. [Desenvolvimento de Aplicativos React](#6-desenvolvimento-de-aplicativos-react)
7. [APIs Headless REST](#7-apis-headless-rest)
8. [Desenvolvimento de Widgets e Aplicações](#8-desenvolvimento-de-widgets-e-aplicações)
9. [Integração com Gerenciamento de Conteúdo](#9-integração-com-gerenciamento-de-conteúdo)
10. [Melhores Práticas e Otimização](#10-melhores-práticas-e-otimização)

---

## 1. Introdução ao Desenvolvimento Front-End no Liferay

### 1.1 O que é o Liferay DXP

O Liferay Digital Experience Platform (DXP) é uma plataforma empresarial para criação de portais digitais, intranets, sites, e aplicações web. É baseada em Java e oferece diversas funcionalidades para desenvolvimento de experiências digitais completas.

### 1.2 Arquitetura Front-End do Liferay

O Liferay DXP adota uma arquitetura modular que permite aos desenvolvedores front-end trabalhar com diversos componentes:

- **Temas**: Controlam a aparência geral do site
- **Fragmentos**: Blocos de construção de páginas
- **Templates de Layout**: Definem a estrutura das páginas
- **Templates de Widget**: Customizam a apresentação de portlets/widgets
- **Aplicações JS/React**: Permitem criar aplicações modernas integradas à plataforma

### 1.3 Ferramentas de Desenvolvimento

Para desenvolvimento front-end no Liferay, você utilizará:

- **Node.js e npm**: Para gerenciamento de pacotes e automação
- **Yeoman e geradores Liferay**: Para scaffolding de projetos
- **Liferay JS Toolkit**: Para desenvolvimento de aplicações JavaScript
- **Blade CLI**: Para desenvolvimento e deploy
- **Liferay Developer Studio/IDE**: Para desenvolvimento integrado

### 1.4 Configuração do Ambiente de Desenvolvimento

```bash
# Instalação do Yeoman e geradores Liferay
npm install -g yo
npm install -g generator-liferay-theme
npm install -g generator-liferay-fragment
npm install -g generator-liferay-js

# Instalação do Blade CLI
curl -L https://raw.githubusercontent.com/liferay/liferay-blade-cli/master/installers/local | sh
```

---

## 2. Branding e Temas

### 2.1 Fundamentos de Temas no Liferay

Um tema no Liferay define a aparência geral do site, incluindo cores, tipografia, espaçamentos e elementos visuais. Os temas são construídos usando:

- HTML
- CSS/Sass
- JavaScript
- FreeMarker Templates (FTL)

### 2.2 Criando seu Primeiro Tema

```bash
# Criando um novo tema
yo liferay-theme

# Importando um tema existente
yo liferay-theme:import
```

### 2.3 Estrutura de um Tema

Um tema Liferay possui a seguinte estrutura de diretórios:

```
my-theme/
├── build.gradle (ou package.json para temas npm)
├── src/
│   ├── css/
│   │   ├── _custom.scss (estilos personalizados)
│   │   └── clay/ (se usar componentes Clay)
│   ├── templates/
│   │   ├── portal_normal.ftl (template principal)
│   │   ├── navigation.ftl
│   │   └── ... (outros templates)
│   ├── images/
│   └── js/
└── liferay-theme.json (configurações do tema)
```

### 2.4 Customização de Temas

#### CSS/Sass

Os temas Liferay usam Sass para estilização. O arquivo principal para customização é o `_custom.scss`:

```scss
/* Exemplo de customização */
$primary: #0b5fff;

/* Override das variáveis Clay/Bootstrap */
$font-family-base: 'Open Sans', sans-serif;

/* Estilos personalizados */
.site-header {
  background-color: $primary;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}
```

#### Templates FreeMarker

Os templates FTL permitem modificar a estrutura HTML do tema:

```html
<#-- portal_normal.ftl - Template principal do tema -->
<!DOCTYPE html>
<html class="${root_css_class}" dir="<@liferay.language key="lang.dir" />" lang="${w3c_language_id}">
<head>
    <title>${the_title} - ${company_name}</title>
    <meta content="initial-scale=1.0, width=device-width" name="viewport" />
    <@liferay_util["include"] page=top_head_include />
</head>
<body class="${css_class}">
    <@liferay_ui["quick-access"] contentId="#main-content" />
    <@liferay_util["include"] page=body_top_include />
    <div class="d-flex flex-column min-vh-100">
        <@liferay.control_menu />
        <div class="d-flex flex-column flex-fill" id="wrapper">
            <#include "${full_templates_path}/header.ftl" />
            <section class="flex-fill" id="content">
                <h2 class="sr-only">${the_title}</h2>
                <#if selectable>
                    <@liferay_util["include"] page=content_include />
                <#else>
                    ${portletDisplay.recycle()}
                    ${portletDisplay.setTitle(the_title)}
                    <@liferay_theme["wrap-portlet"] page="portlet.ftl">
                        <@liferay_util["include"] page=content_include />
                    </@>
                </#if>
            </section>
            <#include "${full_templates_path}/footer.ftl" />
        </div>
    </div>
    <@liferay_util["include"] page=body_bottom_include />
    <@liferay_util["include"] page=bottom_include />
</body>
</html>
```

### 2.5 Temas Responsivos

O Liferay usa o framework Clay (baseado no Bootstrap) para criar interfaces responsivas. Você pode utilizar as classes do Clay para garantir que seu tema funcione em diferentes tamanhos de tela:

```html
<div class="container-fluid">
    <div class="row">
        <div class="col-md-8">Conteúdo principal</div>
        <div class="col-md-4">Barra lateral</div>
    </div>
</div>
```

### 2.6 Deployment de Temas

```bash
# Para temas npm
gulp deploy

# Para temas gradle
blade deploy
```

---

## 3. Fragmentos de Página

### 3.1 O que são Fragmentos

Fragmentos são blocos de construção reutilizáveis para criação de páginas no Liferay. Eles consistem em HTML, CSS e JavaScript que podem ser combinados e configurados pelo usuário final para criar páginas.

### 3.2 Tipos de Fragmentos

- **Seções**: Fragmentos que ocupam toda a largura da página e podem conter componentes
- **Componentes**: Fragmentos menores que podem ser adicionados dentro de seções
- **Coleções**: Agrupamentos de fragmentos relacionados

### 3.3 Criando Fragmentos

```bash
# Iniciar um novo projeto de fragmentos
yo liferay-fragment
```

Estrutura de um fragmento:

```
fragment-name/
├── index.html (estrutura do fragmento)
├── styles.css (estilos específicos)
├── main.js (comportamento JavaScript)
└── configuration.json (configurações editáveis)
```

### 3.4 Fragmentos com Configurações Editáveis

Você pode criar fragmentos com opções configuráveis usando a sintaxe de editable:

```html
<!-- HTML com campos configuráveis -->
<div class="banner" data-lfr-editable-id="banner-color" data-lfr-editable-type="color" style="background-color: #0b5fff;">
    <h1 data-lfr-editable-id="banner-title" data-lfr-editable-type="text">Título Editável</h1>
    <p data-lfr-editable-id="banner-text" data-lfr-editable-type="rich-text">
        Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    </p>
</div>
```

O arquivo `configuration.json` permite definir opções avançadas:

```json
{
  "fieldSets": [
    {
      "fields": [
        {
          "name": "bannerHeight",
          "label": "Altura do Banner",
          "type": "select",
          "typeOptions": {
            "validValues": [
              {"value": "small", "label": "Pequeno"},
              {"value": "medium", "label": "Médio"},
              {"value": "large", "label": "Grande"}
            ]
          },
          "defaultValue": "medium"
        }
      ]
    }
  ]
}
```

### 3.5 Integração com APIs Liferay

Os fragmentos podem acessar dados do Liferay usando JavaScript:

```javascript
// Acessando informações do usuário logado
Liferay.on('allPortletsReady', function() {
    if (Liferay.ThemeDisplay.isSignedIn()) {
        const userName = Liferay.ThemeDisplay.getUserName();
        document.querySelector('.user-greeting').textContent = `Olá, ${userName}!`;
    }
});
```

### 3.6 Exportação e Importação de Fragmentos

```bash
# Exportar fragmentos
gulp export

# Importar fragmentos via interface ou CLI
blade gw buildFragments
```

---

## 4. Templates de Widgets e Apresentação de Conteúdo

### 4.1 Visão Geral dos Templates de Widget

Os Templates de Widget (anteriormente chamados de templates de portlet) permitem customizar a apresentação visual dos widgets/portlets do Liferay sem modificar seu código-fonte.

### 4.2 Tecnologias Suportadas

- **FreeMarker (FTL)**: Template engine padrão
- **Velocity (VM)**: Legado, mas ainda suportado
- **ADT (Application Display Templates)**: Para widgets específicos de conteúdo

### 4.3 Criando um Template para Web Content

```ftl
<#-- Template FreeMarker para Web Content -->
<div class="custom-article">
    <h2>${title.getData()}</h2>
    
    <#if image.getData()?? && image.getData() != "">
        <img src="${image.getData()}" alt="${image.getAttribute("alt")}" />
    </#if>
    
    <div class="article-content">
        ${content.getData()}
    </div>
    
    <#if authorName.getData()?has_content>
        <div class="author-info">
            Por: ${authorName.getData()}
            <#if publishDate.getData()?has_content>
                em ${publishDate.getData()}
            </#if>
        </div>
    </#if>
</div>
```

### 4.4 Templates para Asset Publisher

```ftl
<#-- ADT para Asset Publisher -->
<div class="custom-asset-publisher">
    <#if entries?has_content>
        <div class="card-deck">
            <#list entries as entry>
                <#assign assetRenderer = entry.getAssetRenderer() />
                <#assign viewURL = assetPublisherHelper.getAssetViewURL(renderRequest, renderResponse, entry) />
                
                <div class="card">
                    <#if assetRenderer.getPreviewImageURL(renderRequest)??>
                        <img class="card-img-top" src="${assetRenderer.getPreviewImageURL(renderRequest)}" alt="${entry.getTitle(locale)}">
                    </#if>
                    
                    <div class="card-body">
                        <h5 class="card-title">${entry.getTitle(locale)}</h5>
                        <p class="card-text">${stringUtil.shorten(assetRenderer.getSummary(renderRequest), 100)}</p>
                        <a href="${viewURL}" class="btn btn-primary">Ler mais</a>
                    </div>
                </div>
                
                <#if entry_index % 3 == 2 && entry_has_next>
                    </div><div class="card-deck mt-4">
                </#if>
            </#list>
        </div>
    <#else>
        <div class="alert alert-info">
            Não há conteúdo disponível para exibição.
        </div>
    </#if>
</div>
```

### 4.5 Acessando Serviços Liferay em Templates

```ftl
<#-- Acessando serviços Liferay -->
<#assign serviceContext = staticUtil["com.liferay.portal.kernel.service.ServiceContextThreadLocal"].getServiceContext() />
<#assign httpServletRequest = serviceContext.getRequest() />
<#assign themeDisplay = httpServletRequest.getAttribute("THEME_DISPLAY") />

<#assign userLocalService = serviceLocator.findService("com.liferay.portal.kernel.service.UserLocalService") />
<#assign currentUser = userLocalService.getUser(themeDisplay.getUserId()) />

<div class="user-info">
    <p>Nome: ${currentUser.getFullName()}</p>
    <p>Email: ${currentUser.getEmailAddress()}</p>
</div>
```

---

## 5. Layouts e Templates de Layout

### 5.1 Templates de Layout no Liferay

Os Templates de Layout definem como os widgets/portlets são organizados em uma página. Eles criam a estrutura em que os componentes são posicionados.

### 5.2 Criando um Template de Layout

```bash
# Criando um template de layout com Blade
blade create -t layout-template my-layout
```

Estrutura de um template de layout:

```
my-layout/
├── build.gradle
└── src/main/webapp/
    ├── WEB-INF/
    │   └── liferay-layout-templates.xml
    └── my-layout.ftl
```

Exemplo de um template de layout (my-layout.ftl):

```ftl
<div class="my-custom-layout" id="main-content" role="main">
    <div class="portlet-layout row">
        <div class="col-md-8 portlet-column portlet-column-first" id="column-1">
            ${processor.processColumn("column-1", "portlet-column-content portlet-column-content-first")}
        </div>
        <div class="col-md-4 portlet-column portlet-column-last" id="column-2">
            ${processor.processColumn("column-2", "portlet-column-content portlet-column-content-last")}
        </div>
    </div>
    <div class="portlet-layout row">
        <div class="col-md-4 portlet-column portlet-column-first" id="column-3">
            ${processor.processColumn("column-3", "portlet-column-content portlet-column-content-first")}
        </div>
        <div class="col-md-4 portlet-column" id="column-4">
            ${processor.processColumn("column-4", "portlet-column-content")}
        </div>
        <div class="col-md-4 portlet-column portlet-column-last" id="column-5">
            ${processor.processColumn("column-5", "portlet-column-content portlet-column-content-last")}
        </div>
    </div>
</div>
```

### 5.3 Layouts Responsivos

É importante criar layouts que funcionem bem em diferentes tamanhos de tela:

```ftl
<div class="responsive-layout" id="main-content" role="main">
    <div class="portlet-layout row">
        <!-- Layout desktop: 2 colunas lado a lado -->
        <!-- Layout mobile: 2 colunas empilhadas -->
        <div class="col-lg-6 col-12 portlet-column portlet-column-first" id="column-1">
            ${processor.processColumn("column-1", "portlet-column-content portlet-column-content-first")}
        </div>
        <div class="col-lg-6 col-12 portlet-column portlet-column-last" id="column-2">
            ${processor.processColumn("column-2", "portlet-column-content portlet-column-content-last")}
        </div>
    </div>
    
    <!-- Uma coluna em largura total para todos os dispositivos -->
    <div class="portlet-layout row">
        <div class="col-12 portlet-column" id="column-3">
            ${processor.processColumn("column-3", "portlet-column-content")}
        </div>
    </div>
</div>
```

### 5.4 Layouts Avançados com CSS Grid

Você também pode usar CSS Grid para criar layouts mais complexos:

```ftl
<style>
.grid-layout {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    grid-gap: 20px;
}

.grid-header {
    grid-column: 1 / -1;
}

.grid-sidebar {
    grid-column: 1 / span 3;
}

.grid-content {
    grid-column: 4 / -1;
}

.grid-footer {
    grid-column: 1 / -1;
}

@media (max-width: 768px) {
    .grid-layout {
        grid-template-columns: 1fr;
    }
    
    .grid-sidebar, .grid-content {
        grid-column: 1 / -1;
    }
}
</style>

<div class="grid-layout" id="main-content" role="main">
    <div class="grid-header portlet-column" id="column-1">
        ${processor.processColumn("column-1", "portlet-column-content")}
    </div>
    
    <div class="grid-sidebar portlet-column" id="column-2">
        ${processor.processColumn("column-2", "portlet-column-content")}
    </div>
    
    <div class="grid-content portlet-column" id="column-3">
        ${processor.processColumn("column-3", "portlet-column-content")}
    </div>
    
    <div class="grid-footer portlet-column" id="column-4">
        ${processor.processColumn("column-4", "portlet-column-content")}
    </div>
</div>
```

---

## 6. Desenvolvimento de Aplicativos React

### 6.1 Introdução ao React no Liferay

O Liferay DXP suporta o desenvolvimento de aplicações React modernas que se integram à plataforma. Isso permite criar experiências de usuário ricas e interativas.

### 6.2 Configurando o Ambiente

```bash
# Criando uma aplicação React para Liferay
yo liferay-js
# Selecione "React Widget" quando solicitado
```

### 6.3 Estrutura de uma Aplicação React

```
my-react-app/
├── package.json
├── .npmbundlerrc (configuração para o bundler)
├── src/
│   ├── index.js (ponto de entrada)
│   └── components/
│       └── App.js (componente principal)
```

### 6.4 Exemplo de Aplicação React para Liferay

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';

export default function(elementId) {
    ReactDOM.render(
        <App />,
        document.getElementById(elementId)
    );
}

// App.js
import React, { useState, useEffect } from 'react';
import { LiferayApi } from '../utils/liferay-api';

const App = () => {
    const [userData, setUserData] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        // Obter dados do usuário atual
        LiferayApi.getCurrentUser()
            .then(user => {
                setUserData(user);
                setLoading(false);
            })
            .catch(error => {
                console.error('Error fetching user data:', error);
                setLoading(false);
            });
    }, []);
    
    if (loading) {
        return <div className="loading-spinner">Carregando...</div>;
    }
    
    return (
        <div className="user-profile-app">
            {userData ? (
                <div className="user-card">
                    <div className="user-avatar">
                        <img src={userData.portraitURL} alt={userData.fullName} />
                    </div>
                    <div className="user-info">
                        <h2>{userData.fullName}</h2>
                        <p>{userData.emailAddress}</p>
                        <p>Último acesso: {new Date(userData.lastLoginDate).toLocaleString()}</p>
                    </div>
                </div>
            ) : (
                <div className="error-message">
                    Não foi possível carregar os dados do usuário.
                </div>
            )}
        </div>
    );
};

export default App;
```

### 6.5 Interagindo com Serviços Liferay

```javascript
// utils/liferay-api.js
export const LiferayApi = {
    /**
     * Obtém dados do usuário atual
     */
    getCurrentUser: () => {
        return Liferay.Util.fetch(
            '/o/headless-admin-user/v1.0/my-user-account',
            {
                method: 'GET',
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json'
                }
            }
        )
        .then(response => {
            if (!response.ok) {
                throw new Error('Falha ao obter dados do usuário');
            }
            return response.json();
        });
    },
    
    /**
     * Busca conteúdo estruturado (Web Content)
     */
    getStructuredContent: (siteId, limit = 10) => {
        return Liferay.Util.fetch(
            `/o/headless-delivery/v1.0/sites/${siteId}/structured-contents?pageSize=${limit}`,
            {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            }
        )
        .then(response => {
            if (!response.ok) {
                throw new Error('Falha ao obter conteúdos');
            }
            return response.json();
        });
    }
};
```

### 6.6 Deploy de Aplicações React

```bash
# Build e deploy
npm run build
npm run deploy
```

---

## 7. APIs Headless REST

### 7.1 Visão Geral das APIs Headless

O Liferay DXP oferece APIs REST abrangentes que permitem acessar e manipular conteúdo e funcionalidades de forma headless (desacoplada da interface).

### 7.2 Principais APIs Disponíveis

- **Headless Delivery**: Para consumo de conteúdo (Documentos, Web Content, Blogs, etc.)
- **Headless Admin**: Para administração de usuários, organizações e sites
- **Headless Form**: Para trabalhar com formulários
- **Headless Workflow**: Para gerenciar fluxos de trabalho

### 7.3 Autenticação

```javascript
// Autenticação nas APIs Headless usando Basic Auth
const getAuthHeader = () => {
    const username = 'test@liferay.com';
    const password = 'test';
    const base64Credentials = btoa(`${username}:${password}`);
    return `Basic ${base64Credentials}`;
};

// Autenticação com OAuth 2.0
const getOAuthToken = async () => {
    const clientId = 'your-client-id';
    const clientSecret = 'your-client-secret';
    
    const formData = new FormData();
    formData.append('grant_type', 'client_credentials');
    formData.append('client_id', clientId);
    formData.append('client_secret', clientSecret);
    
    const response = await fetch('/o/oauth2/token', {
        method: 'POST',
        body: formData
    });
    
    const data = await response.json();
    return `Bearer ${data.access_token}`;
};
```

### 7.4 Consumindo Conteúdo via API REST

```javascript
// Exemplo de consumo da API Headless Delivery
const getStructuredContent = async (siteId) => {
    const response = await fetch(
        `/o/headless-delivery/v1.0/sites/${siteId}/structured-contents`,
        {
            method: 'GET',
            headers: {
                'Authorization': await getOAuthToken(),
                'Accept': 'application/json'
            }
        }
    );
    
    if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
    }
    
    return await response.json();
};

// Exemplo de busca de documentos
const getDocuments = async (folderId) => {
    const response = await fetch(
        `/o/headless-delivery/v1.0/document-folders/${folderId}/documents`,
        {
            method: 'GET',
            headers: {
                'Authorization': await getOAuthToken(),
                'Accept': 'application/json'
            }
        }
    );
    
    if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
    }
    
    return await response.json();
};
```

### 7.5 Criação e Atualização de Conteúdo

```javascript
// Criando conteúdo estruturado
const createStructuredContent = async (siteId, contentStructureId, data) => {
    const response = await fetch(
        `/o/headless-delivery/v1.0/sites/${siteId}/structured-contents`,
        {
            method: 'POST',
            headers: {
                'Authorization': await getOAuthToken(),
                'Content-Type': 'application/json',
                'Accept': 'application/json'
            },
            body: JSON.stringify({
                contentStructureId: contentStructureId,
                title: data.title,
                contentFields: data.fields
            })
        }
    );
    
    if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
    }
    
    return await response.json();
};

// Atualizando um conteúdo existente
const updateStructuredContent = async (contentId, data) => {
    const response = await fetch(
        `/o/headless-delivery/v1.0/structured-contents/${contentId}`,
        {
            method: 'PUT',
            headers: {
                'Authorization': await getOAuthToken(),
                'Content-Type': 'application/json',
                'Accept': 'application/json'
            },
            body: JSON.stringify({
                title: data.title,
                contentFields: data.fields
            })
        }
    );
    
    if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
    }
    
    return await response.json();
};
```

### 7.6 GraphQL

Além de REST, o Liferay também oferece suporte a GraphQL:

```javascript
// Exemplo de consulta GraphQL
const getContentViaGraphQL = async (siteId) => {
    const query = `
        query {
            structuredContents(siteKey: "${siteId}") {
                items {
                    id
                    title
                    dateCreated
                    contentFields {
                        name
                        dataType
                        dataValue
                    }
                }
                totalCount
            }
        }
    `;
    
    const response = await fetch('/o/graphql', {
        method: 'POST',
        headers: {
            'Authorization': await getOAuthToken(),
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        },
        body: JSON.stringify({ query })
    });
    
    if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
    }
    
    return await response.json();
};
```

---

## 8. Desenvolvimento de Widgets e Aplicações

### 8.1 Tipos de Widgets no Liferay

- **Portlets JSP**: Widgets tradicionais baseados em Java
- **Portlets MVC**: Widgets usando o padrão MVC do Liferay
- **Widgets JavaScript/React**: Aplicações modernas em JavaScript
- **Widgets usando Templates**: Customização via templates (FTL, VM)

### 8.2 Criando um Portlet MVC

```bash
# Criando um portlet MVC com Blade
blade create -t mvc-portlet my-portlet
```

Estrutura de um portlet MVC:

```
my-portlet/
├── build.gradle
└── src/main/
    ├── java/
    │   └── com/example/portlet/
    │       └── MyPortlet.java
    └── resources/
        ├── META-INF/
        │   └── resources/
        │       ├── css/
        │       │   └── main.scss
        │       ├── js/
        │       │   └── main.js
        │       └── view.jsp
        └── content/
            └── Language.properties
```

### 8.3 Criando um Widget React

```bash
# Criando um widget React
yo liferay-js
```

### 8.4 Portlets Baseados em Templates

Os widgets baseados em templates não requerem desenvolvimento Java e podem ser criados diretamente pela interface do Liferay:

1. Acesse o menu **Site Administration > Content & Data > Web Content**
2. Crie uma estrutura que define os campos
3. Crie um template que define a apresentação
4. Vá para **Site Administration > Configuration > Application Display Templates**
5. Crie um novo ADT para o tipo de widget desejado

### 8.5 Comunicação Entre Widgets

Os widgets do Liferay podem se comunicar usando eventos:

```javascript
// Publicando um evento (widget A)
Liferay.fire('userSelected', {
    userId: 12345,
    userName: 'John Doe'
});

// Escutando um evento (widget B)
Liferay.on('userSelected', function(event) {
    console.log('User selected:', event.userId, event.userName);
    // Atualiza a interface baseado no usuário selecionado
});
```

### 8.6 Integração com Backend

```java
// Exemplo de portlet MVC com serviço de backend
@Component(
    immediate = true,
    property = {
        "com.liferay.portlet.display-category=category.sample",
        "com.liferay.portlet.instanceable=true",
        "javax.portlet.display-name=My User Portlet",
        "javax.portlet.init-param.template-path=/",
        "javax.portlet.init-param.view-template=/view.jsp",
        "javax.portlet.name=myuserportlet",
        "javax.portlet.resource-bundle=content.Language",
        "javax.portlet.security-role-ref=power-user,user"
    },
    service = Portlet.class
)
public class MyUserPortlet extends MVCPortlet {
    
    @Reference
    private UserLocalService _userLocalService;
    
    @Override
    public void render(RenderRequest renderRequest, RenderResponse renderResponse)
            throws IOException, PortletException {
        
        ThemeDisplay themeDisplay = (ThemeDisplay) renderRequest.getAttribute(
            WebKeys.THEME_DISPLAY);
        
        try {
            User user = _userLocalService.getUserById(themeDisplay.getUserId());
            renderRequest.setAttribute("user", user);
        }
        catch (Exception e) {
            _log.error(e);
        }
        
        super.render(renderRequest, renderResponse);
    }
    
    private static final Log _log = LogFactoryUtil.getLog(MyUserPortlet.class);
}
```

```jsp
<%-- view.jsp --%>
<%@ include file="/init.jsp" %>

<c:if test="${not empty user}">
    <div class="user-profile">
        <h2>${user.getFullName()}</h2>
        <p>Email: ${user.getEmailAddress()}</p>
        <p>Data de criação: <fmt:formatDate value="${user.getCreateDate()}" pattern="dd/MM/yyyy" /></p>
    </div>
</c:if>
```

---

## 9. Integração com Gerenciamento de Conteúdo

### 9.1 Tipos de Conteúdo

O Liferay DXP oferece vários tipos de conteúdo que podem ser integrados às suas aplicações front-end:

- **Web Content**: Conteúdo estruturado para páginas
- **Documents & Media**: Arquivos e documentos
- **Blogs**: Artigos de blog
- **Message Boards**: Fóruns de discussão
- **Forms**: Formulários interativos

### 9.2 Exibindo Web Content em Fragmentos

```html
<!-- Fragmento que carrega Web Content específico -->
<div class="featured-content">
    <lfr-editable id="featured-content-id" type="text" default-value="0">
        <!-- ID do conteúdo web será configurado aqui -->
    </lfr-editable>
</div>

<script>
(function() {
    const contentId = fragmentElement.querySelector('[data-lfr-editable-id="featured-content-id"]').textContent.trim();
    
    if (contentId && contentId !== '0') {
        Liferay.Util.fetch(
            `/o/headless-delivery/v1.0/structured-contents/${contentId}`,
            {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            }
        )
        .then(response => response.json())
        .then(data => {
            const container = fragmentElement.querySelector('.featured-content');
            
            // Renderiza o conteúdo
            const title = document.createElement('h2');
            title.textContent = data.title;
            
            const content = document.createElement('div');
            content.innerHTML = data.contentFields.find(field => field.name === 'Content').value.data;
            
            container.innerHTML = '';
            container.appendChild(title);
            container.appendChild(content);
        })
        .catch(error => {
            console.error('Erro ao carregar conteúdo:', error);
        });
    }
})();
</script>
```

### 9.3 Document & Media em Fragmentos

```html
<!-- Fragmento que exibe um documento específico -->
<div class="document-viewer">
    <lfr-editable id="document-id" type="text" default-value="0">
        <!-- ID do documento será configurado aqui -->
    </lfr-editable>
    
    <div class="document-container"></div>
</div>

<script>
(function() {
    const documentId = fragmentElement.querySelector('[data-lfr-editable-id="document-id"]').textContent.trim();
    
    if (documentId && documentId !== '0') {
        Liferay.Util.fetch(
            `/o/headless-delivery/v1.0/documents/${documentId}`,
            {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            }
        )
        .then(response => response.json())
        .then(data => {
            const container = fragmentElement.querySelector('.document-container');
            
            // Renderiza as informações do documento
            const title = document.createElement('h3');
            title.textContent = data.title;
            
            const description = document.createElement('p');
            description.textContent = data.description || 'Sem descrição disponível';
            
            const downloadLink = document.createElement('a');
            downloadLink.href = data.contentUrl;
            downloadLink.textContent = 'Download';
            downloadLink.className = 'btn btn-primary';
            downloadLink.target = '_blank';
            
            container.innerHTML = '';
            container.appendChild(title);
            container.appendChild(description);
            container.appendChild(downloadLink);
            
            // Se for uma imagem, mostrar preview
            if (data.contentUrl && /\.(jpg|jpeg|png|gif)$/i.test(data.title)) {
                const preview = document.createElement('img');
                preview.src = data.contentUrl;
                preview.alt = data.title;
                preview.className = 'img-fluid mt-3';
                container.appendChild(preview);
            }
        })
        .catch(error => {
            console.error('Erro ao carregar documento:', error);
        });
    }
})();
</script>
```

### 9.4 Integrando Forms em Aplicações Front-End

```html
<!-- Fragmento que integra um formulário do Liferay -->
<div class="form-container">
    <lfr-editable id="form-id" type="text" default-value="0">
        <!-- ID do formulário será configurado aqui -->
    </lfr-editable>
</div>

<script>
(function() {
    const formId = fragmentElement.querySelector('[data-lfr-editable-id="form-id"]').textContent.trim();
    
    if (formId && formId !== '0') {
        // Carrega o formulário na página
        Liferay.Loader.require('frontend-js-web/liferay/FormIframe', function(FormIframe) {
            new FormIframe({
                formId: formId,
                namespace: '_com_liferay_dynamic_data_mapping_form_web_portlet_FormPortlet_',
                node: fragmentElement.querySelector('.form-container'),
                url: '/o/dynamic-data-mapping-form/'
            });
        });
    }
})();
</script>
```

---

## 10. Melhores Práticas e Otimização

### 10.1 Performance Front-End

#### Otimização de JavaScript

```javascript
// Carregamento assíncrono de scripts
Liferay.Loader.require(
    'module-name',
    function(Module) {
        // Código que usa o módulo
    }
);

// Debouncing para melhorar performance em eventos
function debounce(func, wait) {
    let timeout;
    return function() {
        const context = this;
        const args = arguments;
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(context, args), wait);
    };
}

// Uso do debounce em um evento de scroll
window.addEventListener('scroll', debounce(function() {
    // Código que responde ao evento de scroll
}, 200));
```

#### Otimização de CSS

```scss
// Usar variables para consistência
$primary-color: #0b5fff;
$border-radius: 4px;

// Evitar seletores aninhados profundos
// Ruim
.container {
    .section {
        .item {
            .title {
                color: $primary-color;
            }
        }
    }
}

// Melhor
.container .item-title {
    color: $primary-color;
}

// Usar BEM para nomenclatura
.card {
    border-radius: $border-radius;
    
    &__header {
        background-color: #f5f5f5;
    }
    
    &__title {
        font-weight: bold;
    }
    
    &--featured {
        border: 2px solid $primary-color;
    }
}
```

### 10.2 Acessibilidade

```html
<!-- Exemplos de boas práticas de acessibilidade -->

<!-- Use landmarks/roles para estrutura semântica -->
<header role="banner">
    <h1>Título do Site</h1>
</header>

<nav role="navigation" aria-label="Menu principal">
    <ul>
        <li><a href="#home">Home</a></li>
        <li><a href="#about">Sobre</a></li>
    </ul>
</nav>

<main role="main">
    <section aria-labelledby="section-heading">
        <h2 id="section-heading">Conteúdo Principal</h2>
        <p>Texto do conteúdo...</p>
    </section>
</main>

<!-- Use aria-labels quando necessário -->
<button aria-label="Fechar" class="close-button">×</button>

<!-- Ofereça alternativas para conteúdo visual -->
<img src="logo.png" alt="Logo da Empresa XYZ">

<!-- Formulários acessíveis -->
<form>
    <div class="form-group">
        <label for="name">Nome:</label>
        <input type="text" id="name" name="name" required aria-required="true">
    </div>
    
    <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" required aria-required="true">
        <div id="email-error" class="error-message" aria-live="polite"></div>
    </div>
    
    <button type="submit">Enviar</button>
</form>
```

### 10.3 Internacionalização (i18n)

```javascript
// Exemplo de internacionalização em fragmentos

// No JavaScript
const messages = {
    en_US: {
        welcome: 'Welcome',
        readMore: 'Read more',
        loading: 'Loading...'
    },
    pt_BR: {
        welcome: 'Bem-vindo',
        readMore: 'Leia mais',
        loading: 'Carregando...'
    },
    es_ES: {
        welcome: 'Bienvenido',
        readMore: 'Leer más',
        loading: 'Cargando...'
    }
};

// Obtém o idioma atual do Liferay
const currentLanguageId = Liferay.ThemeDisplay.getLanguageId();
const currentMessages = messages[currentLanguageId] || messages.en_US;

// Usa a mensagem traduzida
document.querySelector('.welcome-message').textContent = currentMessages.welcome;
```

```ftl
<#-- Exemplo de internacionalização em templates FreeMarker -->
<#assign languageUtil = staticUtil["com.liferay.portal.kernel.language.LanguageUtil"] />

<div class="welcome-section">
    <h2>${languageUtil.get(locale, "welcome")}</h2>
    <p>${languageUtil.get(locale, "welcome-message")}</p>
    <a href="#" class="btn btn-primary">${languageUtil.get(locale, "read-more")}</a>
</div>
```

### 10.4 Testes

```javascript
// Testes para componentes React
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import UserProfile from './UserProfile';

describe('UserProfile component', () => {
    const mockUser = {
        fullName: 'John Doe',
        emailAddress: 'john@example.com',
        lastLoginDate: '2023-01-15T10:30:00Z'
    };
    
    test('renders user information correctly', () => {
        render(<UserProfile user={mockUser} />);
        
        expect(screen.getByText('John Doe')).toBeInTheDocument();
        expect(screen.getByText('john@example.com')).toBeInTheDocument();
        expect(screen.getByText(/15\/01\/2023/)).toBeInTheDocument();
    });
    
    test('shows edit form when edit button is clicked', () => {
        render(<UserProfile user={mockUser} editable={true} />);
        
        // Inicialmente o formulário de edição não está visível
        expect(screen.queryByLabelText('Nome:')).not.toBeInTheDocument();
        
        // Clica no botão de editar
        fireEvent.click(screen.getByText('Editar Perfil'));
        
        // Agora o formulário deve estar visível
        expect(screen.getByLabelText('Nome:')).toBeInTheDocument();
        expect(screen.getByDisplayValue('John Doe')).toBeInTheDocument();
    });
});
```

### 10.5 Segurança Front-End

```javascript
// Práticas de segurança para desenvolvimento front-end

// Sanitização de HTML para prevenir XSS
const sanitizeHTML = (html) => {
    const temp = document.createElement('div');
    temp.textContent = html;
    return temp.innerHTML;
};

// Uso da sanitização
const userContent = '<script>alert("XSS")</script>Hello World';
const safeContent = sanitizeHTML(userContent);
document.querySelector('.user-content').innerHTML = safeContent;

// Validação de entrada
const validateEmail = (email) => {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
};

const form = document.querySelector('form');
form.addEventListener('submit', (e) => {
    const email = document.querySelector('#email').value;
    if (!validateEmail(email)) {
        e.preventDefault();
        document.querySelector('#email-error').textContent = 'Email inválido';
    }
});

// Proteção contra CSRF em requisições AJAX
const token = Liferay.authToken;

fetch('/api/endpoint', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': token
    },
    body: JSON.stringify({ data: 'example' })
});
```

---

Este guia cobre os fundamentos essenciais para desenvolvimento front-end no Liferay DXP. Para informações mais específicas e atualizadas, consulte a documentação oficial do Liferay para sua versão específica da plataforma.
