
# ToLookup vs GroupBy Examples

## `ToLookup` Example

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

// Immediate execution:
// The grouping happens immediately, and the results are stored in an ILookup<TKey, TElement>.
var lookup = numbers.ToLookup(n => n % 2 == 0 ? "Even" : "Odd");

Console.WriteLine("Lookup created:");

foreach (var group in lookup)
{
    Console.WriteLine($"{group.Key}: {string.Join(", ", group)}");
}
```

### Output

```
Odd: 1, 3, 5
Even: 2, 4, 6
```

### Accessing groups by key

`ToLookup` creates an indexed collection, so we can access a group directly using its key:

```csharp
var evenNumbers = lookup["Even"];

Console.WriteLine(string.Join(", ", evenNumbers));
```

### Output

```
2, 4, 6
```

### LINQ operations are supported

Although `ILookup` is read-only (you cannot add or remove items), it still supports LINQ queries:

```csharp
var evenGroups = lookup.Where(g => g.Key == "Even");

var keys = lookup.Select(g => g.Key);
```

---

## `GroupBy` Example

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6 };

// Deferred execution:
// The grouping is not performed until the result is enumerated.
var groups = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");

Console.WriteLine("Enumerating GroupBy:");

foreach (var group in groups)
{
    Console.WriteLine($"{group.Key}: {string.Join(", ", group)}");
}
```

### Output

```
Odd: 1, 3, 5
Even: 2, 4, 6
```

### LINQ operations are supported

`GroupBy` returns an `IEnumerable<IGrouping<TKey, TElement>>`, so we can apply further LINQ operations:

```csharp
var evenGroups = groups.Where(g => g.Key == "Even");

var groupKeys = groups.Select(g => g.Key);
```

However, unlike `ToLookup`, it does not provide direct access by key:

```csharp
// groups["Even"]; // Not supported
```

---

## Main Differences Between `ToLookup` and `GroupBy`

| Feature           | **ToLookup**              | **GroupBy**                                             |
| ----------------- | ------------------------- | ------------------------------------------------------- |
| **Execution**     | Immediate                 | Deferred                                                |
| **Return Type**   | `ILookup<TKey, TElement>` | `IEnumerable<IGrouping<TKey, TElement>>`                |
| **Missing Key**   | Returns an empty sequence  | Search returns `null`/`default`          |
| **Best For**            | Fast lookup by key  | Further LINQ processing before enumeration|
| **Access by Key** | Direct (`lookup["Even"]`)    | Search with LINQ (`groups.FirstOrDefault(g => g.Key == "Even")`) |



## Example showing the difference in execution

```csharp
var numbers = new List<int> { 1, 2, 3, 4 };

var lookup = numbers.ToLookup(n => n % 2 == 0 ? "Even" : "Odd");

var groups = numbers.GroupBy(n => n % 2 == 0 ? "Even" : "Odd");

numbers.Add(6);
```

`ToLookup` will **not see** the new item because it was already created:

```csharp
Console.WriteLine(string.Join(", ", lookup["Even"]));
```

Output:

```
2, 4
```

`GroupBy` is evaluated later, so it can see the updated source when enumerated:

```csharp
foreach (var group in groups)
{
    Console.WriteLine($"{group.Key}: {string.Join(", ", group)}");
}
```

Output:

```
Odd: 1, 3
Even: 2, 4, 6
```



## Summary

* Use **`ToLookup`** when you need a permanent grouped structure and fast access using a key.
* Use **`GroupBy`** when you want deferred execution and further LINQ processing.
* Both support LINQ methods such as `Where`, `Select`, and `OrderBy`.
* The main difference is that `ToLookup` builds the grouping immediately and provides key-based lookup, while `GroupBy` delays the grouping until enumeration.
