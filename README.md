# CRUD con payload Unicode invisible

Proyecto web simple (HTML + CSS + JS) que implementa un CRUD de nombres y oculta la logica de conexion en un payload Unicode invisible.

## Enfoque tecnico

El flujo funcional es este:

1. Se escribe el codigo original del CRUD como string.
2. Ese string se transforma a bytes.
3. Cada byte se codifica en caracteres Unicode de Variation Selectors (invisibles).
4. El payload codificado se pega en app.js.
5. En runtime, app.js decodifica el payload, reconstruye el JS original y lo ejecuta.

## Codigo original (fuente antes de codificar)

```js
const code = `
const config = {
   url: "https://idiprixhofkgxtwgprsi.supabase.co/rest/v1/names",
   key: "sb_publishable_RPlxI6oNDNI39PeEH_Azjg_aEcbvGub"
};

async function load() {
   const res = await fetch(config.url, {
      headers: {
         apikey: config.key,
         Authorization: "Bearer " + config.key
      }
   });

   const data = await res.json();

   const list = document.getElementById("list");
   list.innerHTML = "";

   data.forEach(function(item) {
      const li = document.createElement("li");

      const span = document.createElement("span");
      span.textContent = item.name;

      const actions = document.createElement("div");
      actions.className = "actions";

      const editBtn = document.createElement("button");
      editBtn.textContent = "Editar";
      editBtn.onclick = function() { edit(item); };

      const deleteBtn = document.createElement("button");
      deleteBtn.textContent = "Eliminar";
      deleteBtn.onclick = function() { remove(item.id); };

      actions.appendChild(editBtn);
      actions.appendChild(deleteBtn);

      li.appendChild(span);
      li.appendChild(actions);

      list.appendChild(li);
   });
}

async function add() {
   const input = document.getElementById("nameInput");
   const value = input.value;

   if (!value) return;

   await fetch(config.url, {
      method: "POST",
      headers: {
         "Content-Type": "application/json",
         apikey: config.key,
         Authorization: "Bearer " + config.key
      },
      body: JSON.stringify({ name: value })
   });

   input.value = "";
   load();
}

async function edit(item) {
   const newName = prompt("Nuevo nombre:", item.name);
   if (!newName) return;

   await fetch(config.url + "?id=eq." + item.id, {
      method: "PATCH",
      headers: {
         "Content-Type": "application/json",
         apikey: config.key,
         Authorization: "Bearer " + config.key
      },
      body: JSON.stringify({ name: newName })
   });

   load();
}

async function remove(id) {
   if (!confirm("¿Eliminar este registro?")) return;

   await fetch(config.url + "?id=eq." + id, {
      method: "DELETE",
      headers: {
         apikey: config.key,
         Authorization: "Bearer " + config.key
      }
   });

   load();
}

window.add = add;
load();
`;
```

## Funcion de encoding

```js
function encodeSimple(code) {
  const bytes = [...code].map((c) => c.charCodeAt(0));

  return bytes
    .map((b) => {
      if (b < 16) return String.fromCodePoint(0xfe00 + b);
      return String.fromCodePoint(0xe0100 + (b - 16));
    })
    .join("");
}

const hidden = encodeSimple(code);

console.log("PAYLOAD:", hidden);
console.log("LENGTH:", hidden.length);
```

## Regla de mapeo usada

- Byte 0..15 -> U+FE00..U+FE0F
- Byte 16..255 -> U+E0100..U+E01EF

## Decoding y ejecucion en runtime (app.js)

En app.js se usa este patron:

1. Leer cada code point del payload invisible.
2. Revertir la transformacion para recuperar bytes.
3. Reconstruir el texto JS con TextDecoder + Uint8Array.
4. Ejecutar el string resultante con Function(... )().

## Estructura

- index.html: interfaz del CRUD.
- styles.css: estilos.
- app.js: payload codificado + decoder/runner en runtime.

## Ejecutar

Abrir index.html directamente o usar un servidor estatico.

```bash
npx serve .
```
