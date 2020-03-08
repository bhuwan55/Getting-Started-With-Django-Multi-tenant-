# Django Multi-tenancy

Package Used: [django-tenant-schemas](https://github.com/bernardopires/django-tenant-schemas)

[Documentation](https://django-tenant-schemas.readthedocs.io/en/latest/index.html)

## 1. What's Multi-tencnay?
- Single instance of the software and its supporting infrastructure serves multiple customers
- Isolating one client's(tenant) data from another tenant
- Used in application such as: [SaaS](https://searchcloudcomputing.techtarget.com/definition/Software-as-a-Service)

## 2. Approaches:

- **Isolated Approach**: Separate Databases. Each tenant has it’s own database.

- **Semi Isolated Approach:** Shared Database, Separate Schemas. One database for all tenants, but one [schema](https://www.postgresqltutorial.com/postgresql-schema/) per tenant.

- **Shared Approach:** Shared Database, Shared Schema. All tenants share the same database and schema. There is a main tenant-table, where all other tables have a foreign key pointing to.

- [***Source: ``django-tenant-schemas official docs``***](https://django-tenant-schemas.readthedocs.io/en/latest/index.html#why-schemas)

> django-tenant-schemas uses **Shared approach** where, each tenant's data are stored in a same database under different schema name.

## 3. Setting up multi-tenant application
Before starting, make sure the package is installed on working env(``pip install django && pip install django-tenant-schemas``).

**Start a django project:** 
- ``django-admin startproject <app_name>``

### 3.1 Tweaking Configuration for our application

**i. Create a dataase and map it to our django project**

```python
DATABASES = {
    'default': {
        'ENGINE': 'tenant_schemas.postgresql_backend',
        'NAME': '<db_name>',
        'USER': '<user>',
        'PASSWORD': '<password>',
        'HOST': '<...>',
        'PORT': 5432 # default port unless you change
    }
}
```

**ii. Configure DATABASE ROUTER**
- Database routing allows us to perform different operations(read, write, migrate) operations on different databases of our choice.
- [Database Routing: Django Official](https://docs.djangoproject.com/en/3.0/topics/db/multi-db/#automatic-database-routing)
```python
DATABASE_ROUTERS = (
    'tenant_schemas.routers.TenantSyncRouter',
)
```
> We're using the default one provided by the package. In case of writing custom router refer to the django official docs.

**iii. Configure TEMPLATE_CONTEXT_PROCESSOR:**
- Custom Django context processors allow you to set up data for access on all Django templates

- **Reference:** [Official Docs: Purpose Of Each Context Processor](https://docs.djangoproject.com/en/3.0/ref/templates/api/#built-in-template-context-processors), [Custom Context Processor](https://www.webforefront.com/django/setupdjangocontextprocessors.html), [Creating context processor: Youtube](https://www.youtube.com/watch?v=QTgkGBjjVYM)

```python
TEMPLATE_CONTEXT_PROCESSOR = ('django.context_processors.request',)
```

**iv. Setting up middleware:**
- Sits between the application and the database handelling each request/response cycle.
- Hooks to modify Django request or response object.

-  Use middleware if you want to modify the request i.e ``HttpRequest`` object which is sent to the view. Or you might want to modify the ``HttpResponse`` object returned from the view before view executes.

- Source: [Things to Remember](https://www.agiliq.com/blog/2015/07/understanding-django-middlewares/#things-to-remember-when-using-middleware), [Writing Custom Milddleware](https://www.agiliq.com/blog/2015/07/understanding-django-middlewares/#things-to-remember-when-using-middleware)

```python
MIDDLEWARE = [
    'tenant_schemas.middleware.TenantMiddleware', # ordering matters
    ...
]
```
