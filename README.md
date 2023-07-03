# SOFT DELETE

## Introdução

O famoso CRUD (Create, Read, Update, Delete) é um dos padrões mais utilizados em aplicações web, porém, em alguns casos, o Delete não é tão simples assim.

Temos casos onde o usuário não pode excluir um registro, mas ele pode desativá-lo, ou seja, o registro não será mais exibido, mas ele ainda existe no banco de dados.

---

## Por que não excluir?

Existem vários motivos para não excluir um registro, por exemplo, imagine que você tem um sistema de vendas e um cliente que já comprou um produto não pode ser excluído, pois, se ele for excluído, a venda ficará sem um cliente.

Ou imagine que você tem um sistema de controle de estoque e um produto que já foi vendido não pode ser excluído, pois, se ele for excluído, a venda ficará sem um produto.

Existem vários outros motivos para não excluir um registro, mas o que importa é que, em alguns casos, o registro não pode ser excluído.

---

## O que é Soft Delete?

Soft Delete é uma técnica que consiste em não excluir um registro, mas sim desativá-lo, ou seja, o registro não será mais exibido, mas ele ainda existe no banco de dados.

Seria como se o registro fosse marcado como excluído, mas não fosse excluído de fato.

---

## De que forma o Soft Delete é feito?

Existem várias formas de fazer o Soft Delete, mas a mais comum é adicionar um campo na tabela que será desativada, por exemplo, um campo chamado `status` que pode ter os valores `ativo` ou `inativo`, sendo que o valor `inativo` indica que o registro foi desativado.

Você também pode adicionar um campo chamado `deleted_at` que armazena a data e hora em que o registro foi desativado, sendo que, se o campo `deleted_at` estiver vazio, o registro está ativo, caso contrário, o registro está inativo.

## Exemplo

Vamos supor que você tem uma tabela chamada `clientes` e você quer desativar um cliente, para isso, você pode adicionar um campo chamado `status` que pode ter os valores `ativo` ou `inativo`, sendo que o valor `inativo` indica que o registro foi desativado.

```sql

ALTER TABLE clientes ADD COLUMN status VARCHAR(7) NOT NULL DEFAULT 'ativo';

```

Agora, para desativar um cliente, basta alterar o valor do campo `status` para `inativo`.

```sql

UPDATE clientes SET status = 'inativo' WHERE id = 1;

```

Para verificar se um cliente está ativo ou inativo, basta verificar o valor do campo `status`.

```sql

SELECT * FROM clientes WHERE status = 'ativo';

```

Mas todos concordam que isso é muito trabalhoso, pois, para cada tabela que você quiser desativar um registro, você terá que adicionar um campo chamado `status` e alterar o código para verificar o valor desse campo.

Dessa forma, os frameworks de desenvolvimento web criaram uma forma de fazer o Soft Delete de forma automática, ou seja, você não precisa adicionar um campo chamado `status` em cada tabela, pois o framework faz isso para você.

Como cada framework tem uma forma diferente de fazer o Soft Delete, vamos ver como é feito no Laravel, pois é o framework que eu mais utilizo.

---

## Soft Delete no Laravel

O Laravel utiliza o campo `deleted_at` para armazenar a data e hora em que o registro foi desativado, sendo que, se o campo `deleted_at` estiver vazio, o registro está ativo, caso contrário, o registro está inativo.

Tudo começa com a criação da tabela, onde você deve adicionar o campo `deleted_at` na tabela que será desativada.

Aqui vamos criar uma tabela no banco de dados usando o Artisan do Laravel.

```bash
php artisan make:migration create_clientes_table
```

Agora vamos abrir o arquivo de migração que foi criado e adicionar o campo `deleted_at` na tabela.

```php

Schema::create('clientes', function (Blueprint $table) {
    $table->id();
    $table->string('nome');
    $table->string('email');
    $table->string('telefone');
    $table->timestamps();
    $table->softDeletes();
});

```

Só vou dar uma explicação rápida sobre o código acima, pois não é o foco do artigo.

- O método `id()` cria um campo chamado `id` que é a chave primária da tabela.

- O método `string()` cria um campo do tipo `VARCHAR`.

- O método `timestamps()` cria dois campos chamados `created_at` e `updated_at` que armazenam a data e hora em que o registro foi criado e atualizado.

- O método `softDeletes()` cria um campo chamado `deleted_at` que armazena a data e hora em que o registro foi desativado.

Após adicionar o campo `deleted_at` na tabela, você deve executar a migração para criar a tabela no banco de dados.

```bash

php artisan migrate

```

Agora que a tabela foi criada, vamos criar um Model para ela.

```bash

php artisan make:model Cliente

```

Agora vamos abrir o arquivo do Model e adicionar a trait `SoftDeletes`.

>O que é uma trait?

>Uma trait é um recurso do PHP que permite que você adicione métodos em uma classe sem precisar herdar essa classe. Ou seja, você pode adicionar métodos em uma classe sem precisar herdar essa classe.


```php

use Illuminate\Database\Eloquent\Model;

class Cliente extends Model
{
    use SoftDeletes;
    use HasFactory;

    protected $fillable = [
        'nome',
        'email',
        'telefone',
    ];

}
```
Só vou dar uma explicação rápida sobre o código acima, pois não é todo mundo que conhece o Laravel.

- O método `use SoftDeletes` adiciona a trait `SoftDeletes` no Model.
- O método `use HasFactory` adiciona a trait `HasFactory` no Model, isso é necessário para usar o recurso de Factory do Laravel.
- O método `protected $fillable` indica quais campos podem ser preenchidos em massa, ou seja, quais campos podem ser preenchidos usando o método `create()`.

Agora que o Model foi criado, vamos criar um Controller para ele.

```bash

php artisan make:controller ClienteController --resource

```

O parâmetro `--resource` indica que o Controller será um Resource Controller, ou seja, um Controller que possui os métodos `index`, `create`, `store`, `show`, `edit`, `update` e `destroy`.

Agora vamos abrir o arquivo do Controller e adicionar o Model.

```php

// ...

public function destroy(Cliente $cliente)
{
    $cliente->delete();
    // ...
}

// ...

```

O código acima é o método `destroy()` que é responsável por desativar um registro, ou seja, o código acima desativa um cliente.

Mas como o Laravel sabe que o registro foi desativado?

O Laravel sabe que o registro foi desativado porque o Model `Cliente` utiliza a trait `SoftDeletes`, ou seja, o Laravel sabe que o Model `Cliente` possui um campo chamado `deleted_at` que armazena a data e hora em que o registro foi desativado.

Se nós não tivéssemos utilizado a trait `SoftDeletes`, o Laravel não saberia que o Model `Cliente` possui um campo chamado `deleted_at` que armazena a data e hora em que o registro foi desativado, dessa forma, ao executar o método `delete()`, o Laravel iria excluir o registro.

Agora que o registro foi desativado, vamos verificar se ele está ativo ou inativo.

```php

// ...

public function index()
{
    $clientes = Cliente::all();
    // ...
}

// ...

```

O código acima é o método `index()` que é responsável por exibir todos os registros, ou seja, o código acima exibe todos os clientes.

Mas como o Laravel sabe que o registro está desativado?

O Laravel sabe que o registro está desativado porque o Model `Cliente` utiliza a trait `SoftDeletes`, ou seja, o Laravel sabe que estamos utilizando o recurso de Soft Delete, dessa forma, ao executar o método `all()`, o Laravel não exibe os registros que estão desativados.

Se nós não tivéssemos utilizado a trait `SoftDeletes`, o Laravel não saberia que estamos utilizando o recurso de Soft Delete, dessa forma, ao executar o método `all()`, o Laravel exibiria todos os registros, inclusive os registros que estão desativados.

---

## Conclusão

O Soft Delete é uma técnica que consiste em não excluir um registro, mas sim desativá-lo, ou seja, o registro não será mais exibido, mas ele ainda existe no banco de dados.

O Soft Delete é muito utilizado em sistemas que possuem um sistema de lixeira, ou seja, um sistema que armazena os registros que foram desativados.

Durante o desenvolvimento de um sistema, você deve analisar se o registro pode ser excluído ou não, pois, se o registro não pode ser excluído, você deve utilizar essa técnica.

>Lembre-se que o Soft Delete não é uma regra, ou seja, você não precisa utilizar essa técnica em todos os sistemas que você desenvolver, você deve analisar se o registro pode ser excluído ou não.

>O artigo é só uma introdução ao Soft Delete, ou seja, o artigo não aborda todos os detalhes do Soft Delete, porém, o artigo aborda os detalhes mais básicos e importantes do Soft Delete.

---


