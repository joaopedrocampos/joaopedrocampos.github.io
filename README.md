# Blog

[Blog](https://joaopedrocampos.github.io/) criado usando Jekyll e GitHub Pages

## Objetivo

O objetivo desse site é servir como um "diário" de todos os assuntos que estudar relacionados a engenharia de software em um geral, podendo conter posts de boas práticas de código até design de sistemas.

O principal intuito é servir como um motivador para continuar estudando e para absorver conteúdo de forma mais clara pois para escrever um post sobre um determinado assunto o mesmo deve estar muito claro

## Boas práticas

Todo novo post deve ser criado a partir de um PR para garantir que o conteúdo do mesmo será devidamente revisado ~~vai ser revisado por mim, mas fazer o que ¯\\_(ツ)_/¯~~

## Estrutura do projeto

```bash
--|
  |__posts/ - Posts que devem ser publicados no blog
  |__drafts/ - Posts que são Rascunhos (não serão publicados)
  |__README.md
  |__config.yml
  |__assets/ - Arquivos estáticos, como imagens, para serem usados no blog
```

## Comandos úteis

Para executar o blog localmente, evitando commitar conteúdos desnecessários, pode-se usar o seguinte comando:

```bash
$ bundle exec jekyll serve
```

Para executar o blog localmente e exibir os posts em draft pode-se usar o mesmo comando acima adicionando a flag `--drafts`:

```bash
$ bundle exec jekyll serve --drafts
```

Para mais informações sobre como usar o blog, ver a documentação oficial do [Jekyll](http://jekyllrb.com/docs/)