# Ciando a tela de detalhes do vendedor

Vamos carregar [eager loading](https://docs.microsoft.com/en-us/ef/core/querying/related-data) para carregar objetos relacionados.

Cada vendedor, listado pela view View/Sellers/Index, vai ter um link para a ação "Details", que ainda não existe, sendo assim...

No contolador de vendedor SellersController, vamos criar a ação GET "Details".

```cs

        /*Note que o parametro id é opcional. Para indicar isso, usamos o ponto de interrogação após o int*/
        public IActionResult Details(int? id)
        {
            /*Primeiro vou testar se o id está nulo*/
            if (id == null)
            {
                /*Se estiver nulo, é pq foi efetuada uma requisição de forma indevida,
                nesse caso, iremos retornar uma página de erro padrão.
                Posteriormente vamos personalizar essa página de erro.*/
                return NotFound();
            }

            /*Se o id não estiver nulo, vou buscar meu registro no banco.
            Note que usamos id.value, pois se trata de uma variável que pode
            ter um valor nulo (nulable)*/
            var obj = _sellerService.FindById(id.Value);

            /*Verificar se o id é válido e retornou algum registro.
            Se nenhum vendedor for localizado com esse id, será retornado 
            o valor null do método FindById(int id)*/
            if (obj == null)
            {
                return NotFound();
            }
            return View(obj);
        }

```

Agora vamos criar a view "Details" - View/Sellers/Details
Crie a view sem template e apenas com a opção "Use a layout page" marcada

```html

@model Wallmart.Models.Seller

@{
    ViewData["Title"] = "Details";
}

<div>
    <h4>Seller Details</h4>
    <hr />

    <h2>@ViewData["Title"]</h2>

    <dl class="dl-horizontal">
        <dt>
            @Html.DisplayNameFor(model => model.Name)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.Name)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.Email)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.Email)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.BirthDate)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.BirthDate)
        </dd>
        <dt>
            @Html.DisplayNameFor(model => model.Salary)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.Salary)
        </dd>
    </dl>
 </div>
<div>
    <a asp-action="Edit" asp-route-id="@Model.Id" class="btn btn-default">Edit</a>
    <a asp-action="Index">Back to List</a>


</div>


```

Note que nós não estamos mostrando o nome do departamento do vendedor, vamos adicionar esse campo na view Details.

```html

        <dt>
            @Html.DisplayNameFor(model => model.Department)
        </dt>
        <dd>
            @Html.DisplayFor(model => model.Department.Name)
        </dd>

```

Não apareceu a informação ao rodar o sistema? Isso ocorre, pois o nome do departamento seria um dado de outro objeto e 
nosso método de busca por ID (FindById) está retornando apenas o objeto Seller. 

Para resolver isso e carregar o departamento junto, devemos deixar isso explícito ao EntityFramework por meio de uma instrução que 
indica que o EntityFramework deve fazer o join das tabelas.

Então, no método FindById do SellerService, antes do _.FirstOrDefault_, vamos incluir o trecho "Include(obj => obj.Department)" 
(*não esqueça de importar:* Microsoft.EntityFrameworkCore)

```cs

return _context.Seller.Include(obj => obj.Department).FirstOrDefault(obj => obj.Id == id);
       
```

Você também pode incluir no método FindAll: Include(obj => obj.AnotherObject) 
