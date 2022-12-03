---
title: "101 Entity Framework"
date: 2022-11-21T10:31:42+01:00
draft: false
---

- toList() ouvre une connexion à la bdd, l'énumération aussi mais c'est pas une bonne façon de faire
- use where(s=>s.Name == name).ToList instead of where(s=>s.Name == "Karim").ToList to avoid sql injection
- find return select \* top 1
- orderBy ThenBy => EF take only the last orderBy
- add-migration initial
- script-migration idempotent to generate a script with IF existed condition
- reverse migration, from database to code (power tool)
- .entry to update something already exist in memory , update state / load (include)
- attention for performance using include , try AsSplitQuery to improve it
- attention ti use lazyLoading , to use it : add reference Proxies / add optionsBuilder.useLazyLoadingProxies() / virtual in all entity ; to avoid : count() , data bind grid , no contexte in scope ?
- \_context.ChangeTracker.DebugView.ShortView; too cool
- use .AsNoTracking() to avoid tracking and improve performance (readonly)
- .FromSqlInterpolated($"SomeProcStock '{SomeParameter}$'") or .SqlQuery<YourEntityType>("storedProcedureName",params);
- never use concatenation for raw SQL (sql injection)
- to deal with views =>
  - Migration.Builder(@"Create View ClassName ...")
  - Add DbSet<ClassName>
  - .HasNoKey().ToView(nameOf(ClassName))
- .UseTrackingBehavior(QueryTrackingBehavior.NoTracking) use this in builder.Services.AddDbContext to avoid tracking queries to improve performances (if the context lets you)
- to show a cycle object (Authors.Include(a=>a.Books)) from api => builder.Services.AddControllers().AddJsonOptions(opt=> opt.JsonSerializerOptions.ReferenceHandler.IgnoreCycles );
- for integration tests with real database:
  - make a new DbContextOptionsBuilder<ClassContext>().UseSalServer("sqlserver\*\*\*\*");
  - using new ClassContext(builder.Options)
  - contexte.DataBase.EnsureDelete(); contexte.Database.EnsureCreate();
- for integration tests with memory database:
  - make a new DbContextOptionsBuilder<ClassContext>().IseMemoryDataBase("DataBaseName");
  - using new ClassContext(builder.Options)
- HasConversion<String>() pour stocker l'enum en string.
- using var transaction = \_context.DataBase.BeginTransaction();{..... \_context.SaveChanges(); transaction.Commit()};
- Shadow Properties ? data that's not irrelevant to business logic like : CreatedDate, UpdatedDate ... , it's practical to
  update a ShadowPRop from an overridden SaveChanges method
- use overriding Interceptor to change query with 'option (robust plan)'
