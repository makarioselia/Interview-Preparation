# Inheritance Strategies in EF
Entity Framework supports different inheritance strategies for mapping object-oriented classes to relational database tables. The 3 main strategies are **Table-Per-Hierarchy (TPH)**, **Table-Per-Type (TPT)** and **Table-Per-Concrete Class (TPC)**.

Suppose we have these classes:

```csharp
public class Vehicle
{
    public int Id { get; set; }
    public string Model { get; set; }
}

public class Car : Vehicle
{
    public int NumberOfDoors { get; set; }
}

public class Motorcycle : Vehicle
{
    public bool HasSidecar { get; set; }
}
```

## **1. Table-Per-Hierarchy (TPH) – Default**

In TPH, a **single database table** is used to store data for all classes in the inheritance hierarchy.
### **How It Works**
- A **discriminator column** is added to the table to indicate the type of entity stored in each row.
- All properties of the base class and derived classes are stored as columns in the same table, with nulls for properties not applicable to a specific class.

### **Table (TPH)**

| Id | Model | NumberOfDoors | HasSidecar | Discriminator |
|----|--------|---------------|------------|---------------|
| 1 | Corolla | 4 | NULL | Car |
| 2 | Harley | NULL | False | Motorcycle |

### **Advantages**
- Simple schema with a single table.
- Efficient for querying data, as no joins are needed.

### **Disadvantages**
- Many unused (nullable) columns for derived types.
- Potential for schema complexity with deep hierarchies.

### **Configuration**

You can configure TPH using the default behavior in Entity Framework, which maps inheritance to a single table by default. To explicitly specify TPH:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Vehicle>()
        .HasDiscriminator<string>("Discriminator")
        .HasValue<Vehicle>("Vehicle")
        .HasValue<Car>("Car")
        .HasValue<Motorcycle>("Motorcycle");
}
```



## **2. Table-Per-Type (TPT)**
In TPT, each class in the inheritance hierarchy is mapped to a **separate table** in the database **which contains only the properties of that class and is linked via primary/foreign keys to its ancestors**.

### **How It Works**
- The base class properties are stored in one table.
- Each derived class has its own table containing only its unique properties.
- A join between the base and derived tables is required to retrieve complete data for derived types.

### **Tables (TPT)**

**Vehicles (Base class)**

| Id (PK) | Model |
|----------|--------|
| 1 | Corolla |
| 2 | Harley |

**Cars (Derived class)**

| Id (PK + FK) | NumberOfDoors |
|--------------|---------------|
| 1            | 4             |

**Motorcycles (Derived class)**

| Id (PK + FK) | HasSidecar |
|--------------|------------|
| 2            | False      |

### **Advantages**
- Cleaner table structure with no unused columns.
- Better normalization and alignment with relational design principles.

### **Disadvantages**
- Performance impact due to joins across multiple tables.
- Complex queries when fetching complete data.

### **Configuration**

Starting from EF Core 5.0, TPT can be explicitly configured using the `ToTable` method for each class:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Vehicle>().ToTable("Vehicles");
    modelBuilder.Entity<Car>().ToTable("Cars");
    modelBuilder.Entity<Motorcycle>().ToTable("Motorcycles");
}
```


## **3. Table-Per-Concrete Class (TPC)**

In TPC, each concrete class in the inheritance hierarchy is mapped to its **own table** in the database **which contains all properties of the class and its ancestors**.

### **How It Works**
- Each table includes all the properties of the base class and its derived class.
- There is no table for the base class itself.

### **Tables (TPC)**

**Cars**

| Id | Model | NumberOfDoors |
|----|--------|---------------|
| 1 | Corolla | 4 |

**Motorcycles**

| Id | Model | HasSidecar |
|----|--------|------------|
| 2 | Harley | False |

### **Advantages**
- No joins are needed to query data for specific types.
- Simplifies queries when working with concrete classes.

### **Disadvantages**
- Data duplication for shared properties of the base class.
- Schema updates (e.g., adding properties to the base class) require changes in multiple tables.

### **Configuration**

Starting from EF Core 7.0, TPC can be configured using the `UseTpcMappingStrategy` method. Here's an example:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Vehicle>().UseTpcMappingStrategy();
    modelBuilder.Entity<Car>().ToTable("Cars");
    modelBuilder.Entity<Motorcycle>().ToTable("Motorcycles");
}
```



## **Conclusion**

| Strategy | Tables | Joins | Nullable Columns |  Schema Duplication  |
|----------|--------|-------|------------------|------------------|
| **TPH** | One table | ❌ No | ✅ Yes | ❌ No |
| **TPT** | Base + Derived tables | ✅ Yes | ❌ No | ❌ No |
| **TPC** | One table per concrete class | ❌ No | ❌ No | ✅ Yes (`Model` column in both)|

- **TPH**: Single table, fast querying, but potential for null columns and schema complexity.
- **TPT**: Normalized tables, cleaner schema, but slower queries due to joins.
- **TPC**: Separate tables for each concrete class, avoids null columns and joins, but can lead to data duplication and more complex schema management.
