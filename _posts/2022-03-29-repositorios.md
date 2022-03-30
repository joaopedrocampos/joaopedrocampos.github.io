---
title: "Repositórios!"
date: 2022-03-29T23:40:00-03:00
categories:
  - blog
tags:
  - programming
  - repositorios
---

Repositórios...

Não, não estamos falando daquele projeto de _"Hello World!"_ que você fez em Rust para começar a aprender uma linguagem nova mas que nunca passou disso.

Na realidade estamos falando da camada responsável pela por acessar informações em alguma fonte de dados em projetos que utilizam padrões de arquitetura como [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html).

Vamos nos basear no seguinte modelo de **Clean Architecture**:

![Clean Arch](https://miro.medium.com/max/1400/1*vcnYWWn_zhNk6I30meBaPg.png)

A criação de um _Repositório_ acontece em duas etapas:

- Interface: criada na camada _Domínio_ para garantir que as regras de negócio saibam acessar um dado mas sem conhecer a sua origem (MongoDB, SQL, Memcache, etc);
- Implementação: feita na camada de Dados (ou Infraestrutura) fazendo a comunicação e operações com a fonte de dados escolhida.

A primeira etapa é criar a _Interface_ do _Repositório_. A mesma ficará na nossa camada de _Domínio_ já que lá que estão nossas regras de negócio.

Para facilitar nossa vida vamos criar um exemplo. Imagine que estamos fazendo um Sistema Escolar (sim, muito clichê) e estamos implementando a camada de _Domínio_ de `Aluno`.

```ts
class Aluno {
    private _nome: string;
    private _cpf: string;
    private _email: string;
    private _telefone: string[];

    constructor(nome: string, cpf: string, email: string) {
        this._nome = nome;
        this._cpf = cpf;
        this._email = email;
    }

    public adicionarTelefone(ddd: string, numero: string): void {
        this._telefone.push(`(${ddd}) ${numero}`);
    }

    get cpf(): string {
        return this._cpf;
    }

    get email(): string {
        return this._email;
    }

    get nome(): string {
        return this._nome;
    }
}
```

Agora que temos nossos `Alunos`, precisamos armazêna-los em algum lugar.

Qual o próximo passo?

Criar nossa _Interface_.

Aí que entra o primeiro ponto importante do _Repositório_: ele DEVE estar relacionado às regras de negócio, NÃO às implementações técnicas.

O que isso quer dizer?

No nosso _Repositório_ não temos que implementar métodos `save`, `find`, `list`, etc.

Não, temos que implementar (baseado no nosso exemplo de Sistema Escolar) os métodos `matricular`, `listarMatriculados`, `buscar` e mais quaisquer regras de negócio nosso _Caso de Uso_ possa ter.

Como ficaria nossa _Interface_ nesse exemplo?

```ts
interface IAlunosRepository {
    matricular(aluno: Aluno): void;

    buscar(cpf: string): Aluno;

    listarMatriculados(): Aluno[];
}
```

Dessa forma estamos deixando claras quais regras de negócio nossa aplicação precisa atender, então a camada de _Caso de Uso_ que usar nosso _Repositório_ saberá claramente quais operações pode fazer.

Um GRANDE aliado na hora de aplicar esse padrão é o conceito de _Inversão de Controle_ __(IoC)__. Com __IoC__ toda camada que tiver uma depedência irá implementar a _Interface_ da sua dependência ao invés de implementá-la diretamente.

Como assim?

Suponhamos que nós estamos armazenando nossos Alunos em um cache local (uma lista estática), então teremos nosso _Repositório_ `AlunosRepositoryLocalCache`.

Ao invés de atribuirmos ele diratamente ao construtor do nosso Caso de Uso `Secretaria`, iremos atribuir a sua _Interface_.

```ts
class Secretaria {
    private _repository: IAlunosRepository;

    constructor(repository: IAlunosRepository) {
        this._repository = repository;
    }
}
```

Assim nosso _Caso de Uso_ também nunca irá conhecer a implementação do _Repositório_ em si.

Com isso alterar a fonte de dados se torna muito mais simples já que quem usa o _Repositório_ não conhece isso.

Se combinarmos isso com um __Container__ de _Injeção de Dependência_ sua implementação ficará incrível.

MAS vamos volta ao assunto _Repositório_ pois podemos ter um outro post para falar só de _Injeção de Dependências_...

Agora já temos nossa _Interface_ criada e nosso _Caso de Uso_ usando a mesma só nos resta fazer a implementação do nosso _Repositório_ fazendo as operações definidas na _Interface_.

Para facilitar o exemplo vamos usar um cache local, assim não dependemos de nenhuma lib externa.

```ts
class AlunosRepositoryLocalCache implements IAlunosRepository {
    private matriculados: Aluno[] = [];

    public matricular(aluno: Aluno): void {
        this.matriculados.push(aluno);
    }

    public buscar(cpf: string): Aluno {
        const aluno = this.matriculados.find((aluno) => aluno.cpf.numero === cpf.numero);

        if (!aluno) throw new AlunoNotFound(cpf);

        return aluno;
    }

    public listarMatriculados(): Aluno[] {
        return this.matriculados;
    }
}
```

Um ponto importante que vale ressaltar é o tratamento de excessões com excessões personalizadas para erros esperados, como um registro não encontrado.

No nosso caso criamos o erro `AlunoNotFound`, assim quem for tratar essa possível excessão consegue fazer isso com muito mais facilidade.

```ts
class AlunoNotFound extends Error {
    constructor(cpf: string) {
        super(`Aluno com CPF ${cpf} não encontrado`);
    }
}
```

Pronto, já temos tudo que precisamos e podemos fazer todo o gerenciamento de nossos Alunos no Caso de Uso `Secretaria`.

```ts
class Secretaria {
    private _repository: IAlunosRepository;

    constructor(repository: IAlunosRepository) {
        this._repository = repository;
    }

    public matricularAluno(nome: string, cpf: string, email: string): void {
        const aluno = new AlunoBuilder(nome, cpf, email);

        this._repository.matricular(aluno);
    }

    public listarAlunos(): Aluno[] {
        return this._repository.listarMatriculados();
    }

    public encontrarAluno(cpf: string): Aluno {
        try {
            return this._repository.buscar(cpf);
        } catch (error) {
            if (error instanceof AlunoNotFound) {
                // tratar erro
            }

            throw error;
        }
    }
}
```

Espero que tenha ficado claro o conceito de Repositório e qual as possíveis formas de aplicá-lo. A ideia é explicar em um próximo post como Inversão de Dependência com Inversão de Controle pode nos ajudar na hora de aplicar esse e outros conceitos!

![Bye](https://64.media.tumblr.com/df1aa6a3259f646708d2bc94705c9921/tumblr_pqlxzfwhMI1wo51v0o7_400.gifv)