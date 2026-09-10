# brekan-infrastructure — los descriptores de tenant del parque

**Este repositorio no contiene código.** Solo los descriptores que el Control Plane
escribe cuando un operador da de alta un tenant, y el material sellado que los
acompaña.

## Por qué está separado de `brekan`

El panel de administración necesita **permiso de escritura** sobre el repositorio
que ArgoCD vigila: dar de alta un tenant *es* un commit. Si esos descriptores
vivieran en `brekan`, esa credencial de escritura alcanzaría también el código del
producto — y `master` de `brekan` **no se puede proteger**, porque la protección de
ramas en repositorios privados es de pago.

Separarlo en un repositorio propio es lo que acota el daño: la clave de escritura
del panel no puede tocar nada más que esto. Es la razón entera, y por eso aquí
**nunca** debe entrar código de producto.

## Quién escribe y quién lee

| Quién | Permiso | Cómo |
|---|---|---|
| El panel (`admin/`, en el cluster) | **escritura** | deploy key `control-plane` |
| ArgoCD | **lectura** | deploy key `argocd-lectura` |
| Una persona | lo que le dé su cuenta | revisando un PR |

## Qué hay dentro

```
infrastructure/kubernetes/tenants/generated/<slug>/
├── tenant.yaml         el descriptor: trece secciones, solo referencias a secretos
└── sealed-values.yaml  el material sellado, ciphertext que solo el cluster abre
```

La ruta replica la de `brekan` a propósito: es la que el panel trae por defecto
(`GitOpsProperties.tenantsGeneratedPath`) y la que el `ApplicationSet` ya vigilaba,
así que mover los descriptores aquí no exigió cambiar ni una línea de código.

## El chart NO vive aquí

ArgoCD lee de **dos fuentes**: el chart y su plantilla salen de `brekan` en `master`
—código revisado por PR—, y los descriptores salen de este repositorio. El mecanismo
multi-fuente ya existía: lo introdujo ADR-108 para el archivo de valores sellados.

**Consecuencia que conviene tener presente:** un cambio del chart en `brekan` alcanza
a todos los tenants sin pasar por aquí, y un descriptor de aquí se despliega con el
chart que `master` tenga en ese momento.

## Regla dura

En este repositorio **nunca** se escribe un secreto en claro. Lo que viaja es
ciphertext de `kubeseal`, que solo la clave privada del cluster puede abrir.
