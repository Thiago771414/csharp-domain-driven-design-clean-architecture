# C# Domain-Driven Design Clean Architecture

> A practical case study demonstrating Domain-Driven Design (DDD) using C#, focusing on domain modeling, clean architecture and scalable system design.

![C#](https://img.shields.io/badge/C%23-.NET-blue?style=for-the-badge&logo=csharp)
![DDD](https://img.shields.io/badge/Architecture-DDD-purple?style=for-the-badge)
![Clean Architecture](https://img.shields.io/badge/Pattern-Clean%20Architecture-green?style=for-the-badge)
![Design](https://img.shields.io/badge/Focus-Domain%20Modeling-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Case%20Study-black?style=for-the-badge)

---
## Demo

[![DDD](https://github.com/Thiago771414/imagensProjetos/blob/main/slices/mobile/ddd.png)]


## Overview

Domain-Driven Design (DDD) is an approach that puts the business domain at the center of the system.

Instead of focusing on frameworks or databases, DDD focuses on:

- Business rules
- Domain language
- Model consistency
- Clear boundaries

---

## The Problem

Traditional systems often suffer from:

```text
Anemic domain models
Business logic spread across layers
Tight coupling to infrastructure
Low expressiveness
```
DDD solves this by making the domain the core of the system.

## What is DDD?

DDD is about modeling software based on real-world business concepts.
```text
Domain = business logic + rules
Code = representation of the domain
```
## Ubiquitous Language

DDD introduces a shared language between:

Developers
Business experts
Stakeholders
```text
Cliente → Customer
Pedido → Order
Cancelamento → Cancellation
```
This eliminates ambiguity.

## Core Building Blocks
Entities

Objects with identity.
```csharp
public class Cliente
{
    public int Id { get; set; }
    public string Nome { get; set; }
}
```
## Value Objects

Immutable objects compared by value.
```csharp
public class Endereco
{
    public string Rua { get; set; }
    public string Cidade { get; set; }
}
```
## Aggregates

Group of entities with a root.
```csharp
public class Pedido
{
    public int Id { get; set; }
    public Cliente Cliente { get; set; }
    public ICollection<ItemPedido> Itens { get; set; }
}
```
## Domain Services

Business logic that does not belong to a single entity.
```csharp
public interface IPedidoService
{
    void CriarPedido(Pedido pedido);
    void CancelarPedido(int pedidoId);
}
```
## Repositories

Abstraction for data persistence.
```csharp
public interface IClienteRepository
{
    Cliente ObterPorId(int clienteId);
    void Salvar(Cliente cliente);
}
```
## Domain Events

Represents important events in the system.
```csharp
public class PedidoCriadoEvent
{
    public Pedido Pedido { get; set; }
}
```
## Architecture Layers

DDD is commonly structured in layers:
```text
Domain Layer        → Business rules
Application Layer   → Use cases
Infrastructure      → External systems (DB, APIs)
```
## Example Implementation
Domain Layer
```csharp
namespace MinhaAplicacao.Domain.Models
{
    public class Cliente
    {
        public int Id { get; set; }
        public string Nome { get; set; }
    }

    public class Pedido
    {
        public int Id { get; set; }
        public Cliente Cliente { get; set; }
        public ICollection<ItemPedido> Itens { get; set; }
    }
}
```
## Application Layer
```csharp
public class PedidoService : IPedidoService
{
    private readonly IClienteRepository _clienteRepository;

    public PedidoService(IClienteRepository clienteRepository)
    {
        _clienteRepository = clienteRepository;
    }

    public void CriarPedido(Pedido pedido)
    {
        var cliente = _clienteRepository.ObterPorId(pedido.Cliente.Id);

        cliente.AtualizarStatusCliente("Ativo");

        var evento = new PedidoCriadoEvent { Pedido = pedido };
        EventDispatcher.Publish(evento);
    }
}
```
## Infrastructure Layer
```csharp
public class ClienteRepository : IClienteRepository
{
    public Cliente ObterPorId(int clienteId)
    {
        // database logic
    }

    public void Salvar(Cliente cliente)
    {
        // persistence logic
    }
}
```
## Thin Controllers: HTTP Is Not the Business

In a DDD architecture, controllers should be thin.

The controller is responsible for understanding the HTTP layer, not for making business decisions.

```text
HTTP Request
   ↓
Controller
   ↓
Request DTO / Command
   ↓
Application Service
   ↓
Domain Model / Value Objects
```
A common mistake is allowing HTTP DTOs to travel through the entire system.

This creates coupling between the API contract and the domain model.
```text
Wrong:
Controller → DTO → Application → Domain → Infrastructure

Better:
Controller → DTO/Command → Application → Domain Objects
```
The DTO belongs to the external boundary.

The domain layer should work with entities, aggregates and value objects, not with HTTP request objects.
## Example
Request DTO
```csharp
public class CriarPedidoRequest
{
    public int ClienteId { get; set; }
    public List<ItemPedidoRequest> Itens { get; set; }
}
```
## Controller
```csharp
[ApiController]
[Route("api/pedidos")]
public class PedidoController : ControllerBase
{
    private readonly CriarPedidoUseCase _criarPedidoUseCase;

    public PedidoController(CriarPedidoUseCase criarPedidoUseCase)
    {
        _criarPedidoUseCase = criarPedidoUseCase;
    }

    [HttpPost]
    public IActionResult CriarPedido([FromBody] CriarPedidoRequest request)
    {
        var command = new CriarPedidoCommand(
            request.ClienteId,
            request.Itens.Select(i => new CriarItemPedidoCommand(
                i.ProdutoId,
                i.Quantidade
            )).ToList()
        );

        _criarPedidoUseCase.Execute(command);

        return Created();
    }
}
```
## Application Layer
```csharp
public class CriarPedidoUseCase
{
    public void Execute(CriarPedidoCommand command)
    {
        var clienteId = new ClienteId(command.ClienteId);

        var itens = command.Itens
            .Select(i => new ItemPedido(
                new ProdutoId(i.ProdutoId),
                new Quantidade(i.Quantidade)
            ))
            .ToList();

        var pedido = Pedido.Criar(clienteId, itens);

        // persistir pedido / publicar evento / orquestrar fluxo
    }
}
```
## Key Principle
```text
Controller handles protocol.
Application handles use cases.
Domain handles business rules.
```
The controller should not know how to create a valid business object in depth.

It should only translate the HTTP request into an application command.

The domain model is protected from external contracts such as JSON payloads, route parameters or HTTP-specific structures.

## Architecture Insight
```text
Domain = pure business logic
Application = orchestration
Infrastructure = external concerns
```
## Why DDD Works

DDD improves:
```text
Code readability
Maintainability
Scalability
Business alignment
```
## Real Engineering Benefits

In real-world systems:
```text
Easier evolution of business rules
Clear system boundaries
Independent services (microservices ready)
Better collaboration with business teams
```
## Common Mistakes in DDD

❌ Using DDD for simple CRUD systems
❌ Overengineering the domain
❌ Mixing infrastructure with domain
❌ Ignoring bounded contexts
❌ Not defining aggregates properly

## When to Use DDD

Use DDD when:

Business rules are complex
Domain logic is critical
System will evolve over time
Multiple teams work on the system

## Summary

DDD is not about code.

It is about understanding the business deeply and reflecting it in software.
```text
Good code models the system
Great code models the business
```
```markdown
## Real World Use Cases

- Banking systems
- Order processing systems
- Logistics platforms
- Regulatory systems (ANM-like)
```

> **DTO stands for inbound contract. Value Object is a domain rule. Mixing the two is letting HTTP invade the heart of the system.**

## Author

Thiago Lima
Software Engineer | System Design | Distributed Systems
