# 🌍 Despliegue de una Aplicación Web con Terraform y Azure

En esta práctica, configuramos y desplegamos una aplicación web utilizando **Terraform** para gestionar varios recursos en **Azure**. A continuación, se describe cada paso de la práctica, desde la creación del **Resource Group** hasta el despliegue de la **Function App** y la realización de pruebas.

## 📁 Creación del Resource Group

El primer paso fue crear un **Resource Group**, que es donde todos los recursos estarán organizados dentro de Azure.


```hcl
resource "azurerm_resource_group" "rg" {
  name     = var.name_function
  location = var.location
}
```
Este bloque define el Resource Group en la ubicación especificada.


![](docs/resourceGroup.png)

---
## 🗄️ Creación del Storage Account

Posteriormente, creamos una Storage Account para almacenar los datos necesarios para el funcionamiento de la Function App. Este recurso es necesario para que la aplicación funcione correctamente.

```hcl
resource "azurerm_storage_account" "sa" {
  name                     = var.name_function
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

![](docs/storage.png)

---

## 📊 Creación del Service Plan

El siguiente recurso que desplegamos fue un Service Plan que especifica el nivel de servicio para la Function App. En este caso, utilizamos el plan de consumo.

```hcl
resource "azurerm_service_plan" "sp" {
  name                = var.name_function
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  os_type             = "Windows"
  sku_name            = "Y1"
}

```

![](docs/app.png) 

---
## ⚡ Implementación de la Function App

Luego desplegamos la Function App, donde se alojará y ejecutará el código de la aplicación. Esta se configura para ejecutar funciones bajo demanda a través de solicitudes HTTP.

```hcl
resource "azurerm_windows_function_app" "wfa" {
  name                = var.name_function
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  storage_account_name       = azurerm_storage_account.sa.name
  storage_account_access_key = azurerm_storage_account.sa.primary_access_key
  service_plan_id            = azurerm_service_plan.sp.id

  site_config {
    application_stack {
      node_version = "~18"
    }
  }
}
```

![](docs/function.png) 

---
## 💻 Código

Finalmente, configuramos el código dentro de la Function App y realizamos pruebas. Aquí cargamos un archivo index.js que responde a solicitudes HTTP tanto GET como POST.

```hcl
resource "azurerm_function_app_function" "faf" {
  name            = var.name_function
  function_app_id = azurerm_windows_function_app.wfa.id
  language        = "Javascript"
  file {
    name    = "index.js"
    content = file("example/index.js")
  }
  test_data = jsonencode({
    "name" = "Azure"
  })
  config_json = jsonencode({
    "bindings" : [
      {
        "authLevel" : "anonymous",
        "type" : "httpTrigger",
        "direction" : "in",
        "name" : "req",
        "methods" : ["get", "post"]
      },
      {
        "type" : "http",
        "direction" : "out",
        "name" : "res"
      }
    ]
  })
}
```
![](docs/code.png)

---
## 🌐 URL de la Function App

Finalmente, generamos una URL que nos permite acceder a la aplicación desde un navegador o mediante una llamada HTTP. Esto se logra utilizando el siguiente bloque en output.tf.

```hcl
output "url" {
  value       = azurerm_function_app_function.faf.invocation_url
  sensitive   = false
  description = "description"
}
```
Esta URL es pública y accesible para invocar la función desplegada.

![](docs/url.png)

