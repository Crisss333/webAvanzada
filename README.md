# Laboratorio 1 - Angular, Git, CI/CD y Terraform

Laboratorio evaluado de Ingeniería Web Avanzada (OII436-1). El proyecto implementa un frontend Angular 20, validación continua mediante GitHub Actions y una entrega continua a un ambiente de `staging` simulado con Terraform.

## Integrante

- GitHub: [Crisss333](https://github.com/Crisss333)
- Repositorio: [Crisss333/webAvanzada](https://github.com/Crisss333/webAvanzada)
- Rama de trabajo: `devops/ci-cd`

## Ejecución local

Requisitos: Node.js 20, npm y Terraform 1.5 o superior.

```bash
cd frontend
npm ci
npm test -- --watch=false
npm run build
```

Validación de Terraform, sin aplicar cambios localmente:

```bash
cd infra
terraform fmt -check
terraform init
terraform validate
terraform plan
```

## Respuestas

### 1. ¿Por qué no se recomienda desarrollar directamente sobre `main`?

`main` debe representar una versión estable e integrable. Trabajar en una rama separada permite aislar cambios, revisarlos mediante un Pull Request y exigir que las validaciones automáticas pasen antes de incorporarlos. También facilita la trazabilidad, la colaboración y la reversión de cambios sin comprometer la rama principal.

### 2. ¿Qué problema evita `--skip-git` al crear Angular?

Evita que Angular inicialice un segundo repositorio `.git` dentro de `frontend/`. Un repositorio anidado tendría historial y seguimiento independientes, por lo que el repositorio raíz no administraría correctamente todos los archivos del laboratorio.

### 3. ¿Qué verifica `npm run build`?

Verifica que TypeScript, las plantillas, estilos y dependencias puedan compilarse y empaquetarse correctamente en una versión de producción. Detecta errores de compilación antes de automatizar el mismo proceso en CI; no reemplaza las pruebas unitarias.

### 4. ¿Para qué se revisan `git status` y `git diff --cached` antes de un commit?

Permiten comprobar el alcance exacto del commit. `git status` distingue archivos modificados, preparados y no rastreados; `git diff --cached` muestra el contenido que será registrado. Esta revisión evita omitir archivos necesarios o versionar por accidente secretos, artefactos y cambios ajenos al objetivo del commit.

### 5. ¿Qué evento activa `ci.yml`?

El evento `pull_request` dirigido a la rama `main`. Se ejecuta al crear o actualizar un Pull Request cuya base es `main`.

### 6. ¿Qué representa `ubuntu-latest`?

Es la etiqueta de una imagen de runner hospedado por GitHub Actions. Cada job recibe una máquina virtual limpia con la versión de Ubuntu que GitHub identifica actualmente como estable para esa etiqueta y con herramientas base preinstaladas.

### 7. Orden de las etapas del job `frontend`

1. Obtener el código con `actions/checkout`.
2. Configurar Node.js 20 y la caché de npm.
3. Instalar las dependencias con `npm ci`.
4. Ejecutar las pruebas con `npm test -- --watch=false`.
5. Construir Angular con `npm run build`.

`npm ci` se ejecuta antes porque las pruebas y la compilación requieren las dependencias declaradas. Además, instala exactamente las versiones del `package-lock.json`, lo que hace la ejecución limpia y reproducible.

### 8. ¿Qué etapa falla en el ejercicio controlado y qué ocurre después?

Falla **Ejecutar pruebas**, porque la prueba espera `Título incorrecto` mientras la interfaz muestra `Catálogo de Recursos`. Al devolver un código distinto de cero, el job se detiene y **Construir Angular** queda omitida; por lo tanto, el pipeline completo termina fallando.

### 9. ¿Debe integrarse el Pull Request mientras falla el pipeline?

No. Un pipeline fallido indica que el cambio no supera el quality gate acordado. Integrarlo rompería la confianza en `main` y podría propagar un defecto hacia la entrega. Primero debe corregirse la causa y obtenerse una ejecución exitosa.

### 10. Clasificación de archivos, configuración y secretos

| Elemento | Clasificación | Justificación |
| --- | --- | --- |
| `package.json` | Versionable | Declara scripts y dependencias necesarias para reproducir el proyecto. |
| `API_URL` pública | Variable/configuración | Cambia según el ambiente, pero su valor público no es una credencial. |
| `AWS_REGION` | Variable/configuración | Identifica una región de despliegue y no autentica por sí sola. |
| `DB_PASSWORD` | Secreto/no versionable | Permite autenticarse contra la base de datos. |
| `API_TOKEN` | Secreto/no versionable | Otorga acceso a una API con los permisos asociados al token. |
| `terraform.tfstate` | Secreto/no versionable | Puede contener identificadores, configuración y valores sensibles; debe almacenarse en un backend seguro, no en Git. |

### 11. ¿Por qué no se escriben contraseñas o tokens en workflows o TypeScript?

Porque quedarían expuestos en el historial Git y podrían aparecer en revisiones, forks, artefactos, logs o en el bundle descargable del frontend. Los secretos deben almacenarse en GitHub Secrets e inyectarse únicamente durante la ejecución que los necesita, con permisos mínimos.

### 12. ¿Agregar a `.gitignore` soluciona un secreto ya registrado?

No. `.gitignore` evita seguimientos futuros, pero no elimina el valor del historial. Se debe revocar o rotar inmediatamente la credencial, evaluar el impacto y eliminarla del historial con una herramienta apropiada, coordinando el cambio de historial remoto con el equipo. La rotación es obligatoria porque una copia del secreto pudo haber sido obtenida antes de la limpieza.

### 13. Diferencia entre `terraform validate`, `plan` y `apply`

- `terraform validate` comprueba la sintaxis y coherencia interna de la configuración.
- `terraform plan` calcula y muestra las acciones propuestas para alcanzar el estado declarado, sin ejecutarlas.
- `terraform apply` ejecuta las acciones del plan y actualiza el estado de Terraform.

### 14. ¿Por qué CI usa `pull_request` y CD usa `push` sobre `main`?

CI valida el cambio antes de integrarlo y actúa como puerta de calidad del Pull Request. CD se activa solo cuando un cambio aceptado llega a `main`, de modo que el ambiente de staging reciba únicamente código revisado y validado.

### 15. ¿Qué función cumple Terraform en este CD?

Terraform describe y automatiza de manera reproducible la preparación del staging simulado. El recurso `terraform_data` ejecuta la copia del build Angular hacia `staging/` y expone la ruta como salida. En un escenario real, el mismo enfoque declarativo administraría recursos de infraestructura; aquí no requiere una cuenta cloud.

### 16. ¿Por qué se usa `${{ secrets.DEMO_TOKEN }}`?

Porque GitHub Secrets conserva el valor fuera del repositorio y lo entrega al runner solo durante la ejecución autorizada. Así el workflow referencia el secreto sin escribirlo en YAML ni en el historial, y GitHub puede ocultarlo en los logs. El secreto debe ser ficticio para esta demostración y utilizarse con el alcance mínimo.

## Evidencias

- Pull Request: [devops/ci-cd → main](https://github.com/Crisss333/webAvanzada/pull/1)
- CI exitoso previo al fallo controlado: [ejecución 34127627868](https://github.com/Crisss333/webAvanzada/actions/runs/34127627868)
- CI con fallo controlado: [ejecución 34127788624](https://github.com/Crisss333/webAvanzada/actions/runs/34127788624)
- CI exitoso después de corregir el fallo: [ejecución 34127955064](https://github.com/Crisss333/webAvanzada/actions/runs/34127955064)
- La ejecución de CD se incorporará al completar el merge.

## Seguridad

- Los archivos `.env`, estados de Terraform, variables sensibles y claves privadas están excluidos mediante `.gitignore`.
- `APP_ENV` se almacena como GitHub Actions Variable.
- `DEMO_TOKEN` utiliza un valor ficticio almacenado como GitHub Actions Secret.
- No se utilizan credenciales reales ni servicios cloud.
