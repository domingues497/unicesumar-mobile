# Implementação do AutoRoute (Relato de Implementação)

## Contexto

Neste projeto do catálogo de filmes, a navegação estava sendo feita de forma imperativa com `Navigator.push()` e `Navigator.pop()`. O desafio da aula pediu para refatorar a navegação para usar o pacote AutoRoute, que trabalha com rotas declarativas e (normalmente) gera código para facilitar a navegação tipada.

O objetivo final foi manter a mesma experiência do app, mas organizando a navegação com um router e substituindo o `MaterialApp` por `MaterialApp.router`.

## O que foi feito (passo a passo)

### 1) Dependências adicionadas no pubspec.yaml

No arquivo `pubspec.yaml`, foram adicionadas as dependências necessárias para o AutoRoute funcionar e para o gerador criar as classes de rotas:

- `auto_route` em `dependencies`
- `auto_route_generator` em `dev_dependencies`
- `build_runner` em `dev_dependencies`

Motivo: o AutoRoute depende de geração de código para criar as classes `...Route` que a gente usa ao navegar (por exemplo `DetalhesFilmeRoute`).

### 2) Configuração do router (AppRouter)

No `lib/main.dart`, foi criada a classe:

- `AppRouter extends RootStackRouter`

Ela foi anotada com:

- `@AutoRouterConfig(replaceInRouteName: 'Screen,Route')`

E nela foram declaradas duas rotas:

- rota inicial do catálogo/lista (path `/`)
- rota de detalhes do filme (path `/detalhes`)

Motivo: isso atende o Desafio 1, deixando as rotas centralizadas em um lugar, ao invés de criar `MaterialPageRoute` manualmente.

### 3) Anotações nas telas para o AutoRoute reconhecer

As telas que viraram páginas de rota foram anotadas com:

- `@RoutePage()`

No projeto, as páginas são:

- Tela principal do catálogo: `TelaPrincipalMovieAppScreen`
- Tela de detalhes: `DetalhesFilmeScreen`

Motivo: o gerador do AutoRoute só cria rotas para widgets anotados como páginas.

### 4) Troca de MaterialApp para MaterialApp.router (routerConfig)

No `MainApp`, foi substituído:

- `MaterialApp(...)`

por:

- `MaterialApp.router(...)`

e foi ligado o router com:

- `routerConfig: _appRouter.config(...)`

Como o app carrega a lista de filmes no `main()` antes do `runApp`, foi usado um `deepLinkBuilder` para garantir que a rota inicial receba a lista de filmes como argumento.

Motivo: isso atende o Desafio 2 e garante que a tela inicial continue recebendo `filmes` sem quebrar a inicialização.

### 5) Refatoração da navegação no onTap

Antes, ao tocar em um filme, era feito:

- `Navigator.push(context, MaterialPageRoute(...))`

Depois, foi trocado para:

- `context.router.push(DetalhesFilmeRoute(filme: filme))`

E o voltar (back) na tela de detalhes foi ajustado para:

- `context.router.pop()`

Motivo: isso atende o Desafio 3, usando a navegação moderna do AutoRoute e mantendo passagem de parâmetros de forma tipada.

### 6) Geração do arquivo main.gr.dart

Depois de configurar o router e anotar as páginas, foi executado o build runner para gerar o arquivo:

- `lib/main.gr.dart`

Esse arquivo contém as classes de rotas geradas, como:

- `TelaPrincipalMovieAppRoute`
- `DetalhesFilmeRoute`

Motivo: é esse código gerado que permite navegar com classes e argumentos, ao invés de strings soltas.

## Como validar rapidamente

### Rodar geração de código (quando mexer em rotas)

Na pasta `app`:

```bash
dart run build_runner build
```

### Rodar testes

Na pasta `app`:

```bash
flutter doctor 
flutter pub get
flutter run -d chrome
```

## Resultado esperado

- AutoRoute configurado e funcionando com duas telas (catálogo e detalhes)
- Navegação indo para detalhes ao tocar no item
- Botão de voltar retornando para a lista
- Sem erros de build
