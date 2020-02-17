# .NET Core 3.1 HOW TO on implementing a RESTful API

## Controller
- Create a new class (not a new controller) and inherit from ControllerBase. Inheriting from Controller will support Views, which is out of scope  
- Apply the \[ApiController] attribute: it isn't required, but gives some useful behavior  
- Create an action on the controller, e.g.
```csharp
public IActionResult GetAuthors()
{
}
```
