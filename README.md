 // Centralización de valores para los desplegables
const datosDesplegables = {
  marcas: [
    'Audi', 'BMW', 'Chevrolet', 'Citroën', 'Dacia', 'Fiat', 'Ford', 'Honda',
    'Hyundai', 'Kia', 'Mazda', 'Mercedes-Benz', 'Mini', 'Nissan', 'Opel',
    'Peugeot', 'Renault', 'Seat', 'Skoda', 'Suzuki', 'Toyota', 'Volkswagen', 'Volvo'
  ],
  modelos: {
    'Audi': ['A3', 'Q5'],
    'BMW': ['Serie 3', 'X5'],
    'Chevrolet': ['Cruze', 'Captiva'],
    'Citroën': ['C3', 'C5 Aircross'],
    'Dacia': ['Sandero', 'Duster'],
    'Fiat': ['500', 'Tipo'],
    'Ford': ['Focus', 'Kuga'],
    'Honda': ['Civic', 'CR-V'],
    'Hyundai': ['i30', 'Tucson'],
    'Kia': ['Ceed', 'Sportage'],
    'Mazda': ['Mazda3', 'CX-5'],
    'Mercedes-Benz': ['Clase A', 'GLC'],
    'Mini': ['Cooper', 'Countryman'],
    'Nissan': ['Qashqai', 'Micra'],
    'Opel': ['Corsa', 'Astra'],
    'Peugeot': ['208', '3008'],
    'Renault': ['Clio', 'Captur'],
    'Seat': ['Ibiza', 'León'],
    'Skoda': ['Fabia', 'Octavia'],
    'Suzuki': ['Swift', 'Vitara'],
    'Toyota': ['Yaris', 'Corolla'],
    'Volkswagen': ['Golf', 'Tiguan'],
    'Volvo': ['XC40', 'S60']
  },
  familias: [], // Inicializado vacío para actualizar dinámicamente
  frecuencias: ['434 MHz ASK', '434 MHz FSK', '868 MHz FSK', 'SIN ESPECIFICAR'],
  tiposMando: ['Con llave', 'Sin llave', 'Tarjeta'],
  tiposPerfil: ['1', '2', '3'],
  transponders: ['Tipo 1', 'Tipo 2', 'Tipo 3'],
  botones: ['1', '2', '3', '4']
};

// Array para almacenar productos (se cargará desde localStorage si existe)
let productos = JSON.parse(localStorage.getItem('productos')) || [];

// Variable global para rastrear si estamos editando un producto
let productoEnEdicion = null;

// Referencias al modal y formulario
const modal = document.getElementById('productModal');
const form = document.getElementById('productForm');
const marcaSelect = document.getElementById('marca');
const modeloSelect = document.getElementById('modelo');
const familiaSelect = document.getElementById('familia');
const filterMarca = document.getElementById('filterMarca');
const filterModelo = document.getElementById('filterModelo');
const filterFamilia = document.getElementById('filterFamilia');
const filterAnio = document.getElementById('filterAnio');

// Función para abrir el modal y preparar el formulario
document.getElementById('addProduct').addEventListener('click', () => {
  modal.style.display = 'flex'; // Mostrar el modal
  form.reset(); // Limpiar el formulario
  productoEnEdicion = null; // Resetear la edición
  modeloSelect.innerHTML = `<option value="">Seleccionar Modelo</option>`; // Vaciar el selector de modelos
});

// Función para cerrar el modal
document.getElementById('closeModal').addEventListener('click', () => {
  modal.style.display = 'none'; // Ocultar el modal
});

// Validación del campo de años
function validarAnio(anio) {
  if (!anio) return false;

  if (anio.includes("-")) {
    const [inicio, fin] = anio.split("-").map(n => parseInt(n.trim()));
    return !isNaN(inicio) && !isNaN(fin) && inicio <= fin;
  } else {
    const year = parseInt(anio.trim());
    return !isNaN(year);
  }
}

// Función para verificar si un año está dentro de un rango o es igual
function anioEnRango(anio, rango) {
  if (!rango.includes("-")) {
    return parseInt(anio) === parseInt(rango);
  }
  const [inicio, fin] = rango.split("-").map(n => parseInt(n.trim()));
  anio = parseInt(anio.trim());
  return anio >= inicio && anio <= fin;
}

// Función para actualizar desplegables dinámicamente
function actualizarDesplegables() {
  const opcionesMarcas = datosDesplegables.marcas.map(marca => `<option value="${marca}">${marca}</option>`).join('');
  marcaSelect.innerHTML = `<option value="">Seleccionar Marca</option>` + opcionesMarcas;
  filterMarca.innerHTML = `<option value="">Marca</option>` + opcionesMarcas;

  actualizarFamilias(); // Asegurar que las familias también se actualicen
}

// Actualizar desplegable de familias dinámicamente
function actualizarFamilias() {
  const opcionesFamilias = datosDesplegables.familias.map(familia => `<option value="${familia}">${familia}</option>`).join('');
  filterFamilia.innerHTML = `<option value="">Familia</option>` + opcionesFamilias;
  familiaSelect.innerHTML = `<option value="">Seleccionar Familia</option>` + opcionesFamilias;
}

// Actualización dinámica de modelos según la marca seleccionada
filterMarca.addEventListener('change', () => {
  const marcaSeleccionada = filterMarca.value;
  const modelos = datosDesplegables.modelos[marcaSeleccionada] || [];
  filterModelo.innerHTML = `<option value="">Modelo</option>` + modelos.map(modelo => `<option value="${modelo}">${modelo}</option>`).join('');
});

// Actualización dinámica de modelos al seleccionar una marca en el formulario
marcaSelect.addEventListener('change', () => {
  const marcaSeleccionada = marcaSelect.value;
  const modelos = datosDesplegables.modelos[marcaSeleccionada] || [];
  modeloSelect.innerHTML = `<option value="">Seleccionar Modelo</option>` + modelos.map(modelo => `<option value="${modelo}">${modelo}</option>`).join('');
});

// Función para filtrar productos
function filtrarProductos() {
  const familia = filterFamilia.value.trim();
  const marca = filterMarca.value.trim();
  const modelo = filterModelo.value.trim();
  const anio = filterAnio.value.trim();
  const productosFiltrados = productos.filter(producto => {
    const familiaCoincide = !familia || producto.familia === familia;
    const marcaCoincide = !marca || producto.marca === marca;
    const modeloCoincide = !modelo || producto.modelo === modelo;
    const anioCoincide = !anio || anioEnRango(anio, producto.anio);
    return familiaCoincide && marcaCoincide && modeloCoincide && anioCoincide;
  });
  renderTabla(productosFiltrados);
}

// Asignar eventos a los filtros
filterFamilia.addEventListener('change', filtrarProductos);
filterMarca.addEventListener('change', filtrarProductos);
filterModelo.addEventListener('change', filtrarProductos);
filterAnio.addEventListener('input', filtrarProductos);

// Guardar producto al enviar el formulario
form.addEventListener('submit', async event => {
  event.preventDefault(); // Evitar recarga de la página

  const anio = document.getElementById('anio').value.trim();
  if (!validarAnio(anio)) {
    alert('Por favor, introduce un año válido (por ejemplo: "2020" o "2000-2010").');
    return;
  }

  // Convertir imagen a Base64 (si se seleccionó una)
  const fotoInput = document.getElementById('foto');
  let fotoBase64 = productoEnEdicion !== null ? productos[productoEnEdicion].foto : '';
  if (fotoInput.files[0]) {
    fotoBase64 = await convertirImagenABase64(fotoInput.files[0]);
  }

  const nuevoProducto = {
    nombre: document.getElementById('nombre').value.trim(),
    marca: document.getElementById('marca').value.trim(),
    modelo: document.getElementById('modelo').value.trim(),
    anio: anio,
    referencia: document.getElementById('referencia').value.trim(),
    foto: fotoBase64,
    obd: document.getElementById('obd').value.trim(),
    perfil: document.getElementById('perfil').value.trim(),
    frecuencia: document.getElementById('frecuencia').value.trim(),
    botones: document.getElementById('botones').value.trim(),
    mando: document.getElementById('mando').value.trim(),
    transponder: document.getElementById('transponder').value.trim(),
    precio: parseFloat(document.getElementById('precio').value).toFixed(2),
    familia: document.getElementById('familia').value.trim(),
  };

  // Añadir familia si es nueva
  if (!datosDesplegables.familias.includes(nuevoProducto.familia)) {
    datosDesplegables.familias.push(nuevoProducto.familia);
    actualizarFamilias();
  }

  // Guardar o editar producto
  if (productoEnEdicion !== null) {
    productos[productoEnEdicion] = nuevoProducto;
  } else {
    productos.push(nuevoProducto);
  }

  // Guardar en localStorage y actualizar tabla
  localStorage.setItem('productos', JSON.stringify(productos));
  renderTabla(productos);
  modal.style.display = 'none'; // Cerrar el modal
  productoEnEdicion = null; // Resetear edición
});

// Función para convertir imagen a Base64
function convertirImagenABase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error);
    reader.readAsDataURL(file);
  });
}

// Renderizar la tabla de productos
function renderTabla(productosFiltrados) {
  const tabla = document.querySelector('table tbody');
  tabla.innerHTML = productosFiltrados.map((producto, index) => `
    <tr>
      <td><img src="${producto.foto}" alt="Foto del producto" class="product-photo"></td>
      <td>${producto.nombre}</td>
      <td>${producto.marca}</td>
      <td>${producto.modelo}</td>
      <td>${producto.anio}</td>
      <td>${producto.referencia}</td>
      <td>${producto.obd}</td>
      <td>${producto.perfil}</td>
      <td>${producto.frecuencia}</td>
      <td>${producto.botones}</td>
      <td>${producto.mando}</td>
      <td>${producto.transponder}</td>
      <td>${producto.precio}</td>
      <td>${producto.familia}</td>
      <td>
        <button class="btn edit-btn green" onclick="editarProducto(${index})">Editar</button>
        <button class="btn delete-btn red" onclick="eliminarProducto(${index})">Eliminar</button>
      </td>
    </tr>
  `).join('');
}

// Inicializar los desplegables al cargar la página
document.addEventListener('DOMContentLoaded', () => {
  actualizarDesplegables();
  renderTabla(productos);
})

