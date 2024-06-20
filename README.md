# Criando-uma-API-Utilizando-C#

Para criar uma API robusta utilizando C# e .NET, seguindo os passos desde a configuração do ambiente até a implantação, incluindo autenticação, documentação e otimização, vamos detalhar cada etapa. Vamos utilizar o framework ASP.NET Core para construir a API.

### Descrição do Projeto:
Desenvolver uma API funcional e escalável usando C# e ASP.NET Core, abordando desde a configuração inicial até a implantação, incluindo práticas de autenticação e documentação.

### Passos para Implementação:

#### Passo 1: Configuração do Ambiente
Certifique-se de ter o ambiente de desenvolvimento configurado com:
- Visual Studio ou Visual Studio Code
- .NET SDK instalado (disponível em [dot.net](https://dot.net))

#### Passo 2: Criação do Projeto
Abra o Visual Studio ou o Visual Studio Code e crie um novo projeto ASP.NET Core Web API.

#### Passo 3: Implementação da API
Vamos criar uma API simples para gerenciar produtos. Vamos utilizar Entity Framework Core para acesso ao banco de dados (SQL Server).

1. **Definindo um modelo de Produto:**
   - Crie uma classe `Produto` para representar os dados do produto.

```csharp
// Models/Produto.cs
public class Produto
{
    public int Id { get; set; }
    public string Nome { get; set; }
    public decimal Preco { get; set; }
}
```

2. **Configuração do DbContext:**
   - Configure o Entity Framework Core para gerenciar o acesso ao banco de dados.

```csharp
// Data/AppDbContext.cs
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Produto> Produtos { get; set; }
}
```

3. **Criação dos Controladores:**
   - Crie um controlador `ProdutosController` para gerenciar operações CRUD para produtos.

```csharp
// Controllers/ProdutosController.cs
[ApiController]
[Route("api/[controller]")]
public class ProdutosController : ControllerBase
{
    private readonly AppDbContext _context;

    public ProdutosController(AppDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public async Task<ActionResult<IEnumerable<Produto>>> GetProdutos()
    {
        return await _context.Produtos.ToListAsync();
    }

    [HttpPost]
    public async Task<ActionResult<Produto>> PostProduto(Produto produto)
    {
        _context.Produtos.Add(produto);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetProduto), new { id = produto.Id }, produto);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Produto>> GetProduto(int id)
    {
        var produto = await _context.Produtos.FindAsync(id);

        if (produto == null)
        {
            return NotFound();
        }

        return produto;
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> PutProduto(int id, Produto produto)
    {
        if (id != produto.Id)
        {
            return BadRequest();
        }

        _context.Entry(produto).State = EntityState.Modified;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!ProdutoExists(id))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }

        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduto(int id)
    {
        var produto = await _context.Produtos.FindAsync(id);
        if (produto == null)
        {
            return NotFound();
        }

        _context.Produtos.Remove(produto);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool ProdutoExists(int id)
    {
        return _context.Produtos.Any(e => e.Id == id);
    }
}
```

4. **Configuração do Serviço e Middleware:**
   - Configure o serviço de banco de dados e middleware de roteamento na classe `Startup.cs`.

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<AppDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddControllers();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

#### Passo 4: Implantação e Documentação
Para a implantação da API, você pode publicá-la em plataformas como Azure, AWS, ou qualquer provedor de nuvem. Para a documentação, você pode usar o Swagger, uma ferramenta que gera automaticamente a documentação da API a partir do código.

1. **Instalação do Swagger:**
   - Instale o pacote NuGet `Swashbuckle.AspNetCore` para integrar o Swagger à sua API.

```bash
dotnet add package Swashbuckle.AspNetCore
```

2. **Configuração do Swagger:**
   - Configure o Swagger no método `ConfigureServices` em `Startup.cs`.

```csharp
// Startup.cs
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Minha API", Version = "v1" });
});
```

3. **Habilitando o Middleware do Swagger:**
   - Adicione o middleware do Swagger no método `Configure` em `Startup.cs`.

```csharp
// Startup.cs
app.UseSwagger();
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Minha API V1");
});
```

### Executando e Testando a API:
- Compile e execute o projeto. Você poderá testar a API usando ferramentas como Postman ou o próprio Swagger UI, acessível em `http://localhost:<porta>/swagger`.

### Conclusão:
Este projeto prático proporciona uma base sólida para desenvolvimento de APIs em C# e ASP.NET Core, cobrindo desde a configuração inicial até a implementação de operações CRUD básicas e documentação com Swagger. Podemos expandir este projeto adicionando mais funcionalidades, como autenticação com JWT, validação de entrada, entre outros recursos avançados de APIs RESTful.
