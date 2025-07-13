# 🧪 Mocked Controller Test — What Actually Runs?

```
public class UserService : IUserService
{
    private readonly IUserRepository _repo;
    private readonly IUnitOfWork _unitOfWork;

    public UserService(IUserRepository repo, IUnitOfWork unitOfWork)
    {
        _repo = repo;
        _unitOfWork = unitOfWork;
    }

    public void AddUser(AddUserDto dto)
    {
        var user = User.Create(dto.Name, dto.Email);
        _repo.Add(user);
        _unitOfWork.Commit();
    }
}
```


Assume you have the following controller:

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Application;

namespace EndpointApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class UsersController : ControllerBase
    {
        private readonly UserService _service;

        public UsersController(UserService service)
        {
            _service = service;
        }

        [HttpPost]
        public IActionResult Add(AddUserDto dto)
        {
            _service.AddUser(dto);
            return Ok();
        }
    }
}
```

Now imagine this test:

```csharp
[Fact]
public void Add_Should_Call_Service()
{
    var mockService = Substitute.For<IUserService>();
    var controller = new UsersController(mockService);

    var dto = new AddUserDto { Name = "Ali", Email = "ali@example.com" };
    var result = controller.Add(dto);

    result.Should().BeOfType<OkResult>();
    mockService.Received().AddUser(dto);
}
```

---

## ✅ What gets executed?

- `controller.Add(dto)` is **actually executed**.
- Inside that method, `_service.AddUser(dto)` is **called**, but not really executed.
  - Why? Because `mockService` is a fake (substitute), created with `NSubstitute`.

### 🔍 What happens with `mockService.AddUser(dto)`?

- The method is *called*, but its internal logic is not executed.
- It’s just **recorded** so we can verify that it was called later with `.Received()`.

---

## ✅ Summary

| Part | Executed? | Explanation |
|------|-----------|-------------|
| `UsersController.Add(dto)` | ✅ Yes | The real method is executed |
| `_service.AddUser(dto)`   | ❌ No  | Called, but it's a mocked dependency |
| `mockService.Received().AddUser(dto)` | 🔍 Just verifies the call | Doesn't run any logic |

This is typical of a **unit test for the controller**, where we don't test the logic of the service itself.

If you want to write an integration test that executes the actual service logic, let me know!


---

## 🔁 Example 2: Unit Test of `UserService.AddUser` (Real Call, Mocked Dependencies)

Let's consider this real service implementation:

```csharp
public class UserService : IUserService
{
    private readonly IUserRepository _repo;
    private readonly IUnitOfWork _unitOfWork;

    public UserService(IUserRepository repo, IUnitOfWork unitOfWork)
    {
        _repo = repo;
        _unitOfWork = unitOfWork;
    }

    public void AddUser(AddUserDto dto)
    {
        var user = User.Create(dto.Name, dto.Email);
        _repo.Add(user);
        _unitOfWork.Commit();
    }
}
```

And now the unit test:

```csharp
[Fact]
public void AddUser_Should_AddUser_And_Commit()
{
    var repo = Substitute.For<IUserRepository>();
    var uow = Substitute.For<IUnitOfWork>();
    var service = new UserService(repo, uow);

    var dto = new AddUserDto { Name = "Ali", Email = "ali@example.com" };
    service.AddUser(dto); // ✅ This method runs for real

    repo.Received().Add(Arg.Is<User>(u => u.Name == "Ali" && u.Email == "ali@example.com")); // ❌ Only recorded
    uow.Received().Commit(); // ❌ Only recorded
}
```

### ✅ What really runs:
- `UserService.AddUser(...)`: Yes, real code runs
- `User.Create(...)`: Yes, domain logic runs
- `_repo.Add(...)`: No logic runs (mocked)
- `_unitOfWork.Commit()`: No logic runs (mocked)

This test **verifies behavior** by checking **interactions** with mocked dependencies, while letting the actual service logic run.
