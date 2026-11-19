# Billetera Digital — Angular (Caso 2, T1)

Curso **Soluciones Web y Aplicaciones Distribuidas** — UPN, 2026-5.

Billetera digital construida con **dos componentes** que se comunican entre padre e
hijo. El estado vive en el componente padre. No hay backend, servicios, enrutamiento
ni formularios reactivos.

## Versiones utilizadas

| Herramienta | Versión |
|---|---|
| Node.js | 22.22 (compatible: ^20.19, ^22.12 o ≥ 24) |
| npm | 9.2 o superior |
| Angular / Angular CLI | 21.2 |
| TypeScript | 5.9 |
| Bootstrap | 5.3.9 (instalado con npm) |
| Vitest | 4 (pruebas unitarias) |
| IDE | Visual Studio Code + extensión *Angular Language Service* |

## Instalar y ejecutar

```bash
# 1. Instalar dependencias (usa package-lock.json)
npm install

# 2. Ejecutar en modo desarrollo
npm start          # equivalente a: npx ng serve
```

Abrir <http://localhost:4200/>.

Otros comandos:

```bash
npm test                    # pruebas unitarias (incluyen las pruebas de la rúbrica)
npx ng test --watch=false   # pruebas una sola vez, sin modo observación
npm run build               # build de producción en dist/billetera-angular/
```

> No hace falta instalar Angular CLI globalmente: `npm start` y `npx ng ...` usan la
> versión local del proyecto.
>
> **Si `npm install` falla con `Cannot read properties of null (reading 'edgesOut')`**:
> es un bug de npm 9.2, la versión que trae el paquete de Ubuntu, que aparece si
> falta `package-lock.json`. Puedes usar `npx -y npm@11 install` o actualizar Node
> desde <https://nodejs.org/>.

## Estructura

```
src/app/
├── app.ts / app.html                    AppComponent: solo contenedor (navbar + <app-billetera>)
├── movimiento.model.ts                  tipos: TipoMovimiento, SolicitudMovimiento, Movimiento
├── billetera/                           ← PADRE (dueño del estado)
│   ├── billetera.component.ts
│   └── billetera.component.html
└── nuevo-movimiento/                    ← HIJO (formulario + validación)
    ├── nuevo-movimiento.component.ts
    └── nuevo-movimiento.component.html
```

## Comunicación padre ↔ hijo

```
BilleteraComponent (padre)
  saldo = signal(250)
  movimientos = signal<Movimiento[]>([])
      │
      │  [saldo]="saldo()"                          input()
      ▼
NuevoMovimientoComponent (hijo)
  saldo = input.required<number>()
  movimientoValido = output<SolicitudMovimiento>()
  valida monto y saldo → @if muestra errores, [disabled] bloquea el botón
      │
      │  (movimientoValido)="registrar($event)"     output()
      ▼
padre.registrar(): actualiza el saldo e inserta el movimiento AL INICIO del historial
```

| Concepto (Sesión 2) | Dónde se usa |
|---|---|
| Componentes generados con la CLI | `ng generate component billetera --type=component` (y `nuevo-movimiento`) |
| Interpolación | `S/ {{ saldo() \| number: '1.2-2' }}` en ambos componentes |
| Property binding | `[saldo]`, `[disabled]`, `[class.is-invalid]`, `[class.text-bg-success]` |
| `@for` | historial en el padre (`track m.id`) |
| `@if` | "Aún no hay movimientos" (padre); errores de validación (hijo) |
| `input()` / `output()` | saldo hacia el hijo; solicitud válida hacia el padre |
| Bootstrap | `card`, `list-group`, `badge`, `alert`, `form-select`, `form-control`, `btn` (sin CSS propio) |

### Reglas de validación (en el hijo)

- El monto es obligatorio y debe ser mayor que cero, con máximo 2 decimales.
- Un **gasto** no puede superar el saldo recibido por `input()`. Gastar el saldo
  exacto sí está permitido.
- Si algo no se cumple, el botón queda deshabilitado (`[disabled]`) y nunca se
  emite el `output()`.
- El padre vuelve a verificar antes de aplicar, como respaldo. Así nunca deja el
  saldo negativo.

## Pruebas funcionales (rúbrica)

Recargar la página (F5) reinicia la aplicación al estado inicial.

| # | Acción | Resultado esperado |
|---|---|---|
| 1 | Abrir la aplicación | Saldo S/ 250.00; "Aún no hay movimientos" |
| 2 | Registrar recarga de S/ 100.00 | Saldo S/ 350.00; recarga en el historial |
| 3 | Registrar gasto de S/ 50.00 | Saldo S/ 300.00; el gasto aparece antes que la recarga |
| 4 | Intentar gasto de S/ 400.00 | "Saldo insuficiente para realizar el gasto"; botón deshabilitado; saldo sigue en S/ 300.00 |
| 5 | Monto 0, negativo o vacío | Botón deshabilitado (y mensaje si es 0 o negativo); nada cambia |

Estas mismas pruebas están automatizadas en
`src/app/billetera/billetera.component.spec.ts` y se ejecutan con `npm test`.
