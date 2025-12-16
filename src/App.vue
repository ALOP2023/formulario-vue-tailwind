<script setup>
import { ref, onMounted } from 'vue'

const currentStep = ref(1)
const totalSteps = 3

// Datos del formulario
const formData = ref({
  // Paso 1
  pais: '',
  genero: '',
  primerNombre: '',
  segundoNombre: '',
  fechaNacimiento: '',
  tipoDocumento: '',
  numeroDocumento: '',
  fotoDocumentoFrente: null,
  fotoDocumentoReverso: null,
  // Paso 2
  correoElectronico: '',
  contrasena: '',
  confirmacionContrasena: '',
  numeroTelefono: '',
  numeroCelular: '',
  // Paso 3
  direccionResidencia: '',
  codigoPostal: ''
})

// Errores de validación
const errors = ref({
  // Paso 1
  fechaNacimiento: '',
  numeroDocumento: '',
  fotoDocumentoFrente: '',
  fotoDocumentoReverso: '',
  // Paso 2
  correoElectronico: '',
  contrasena: '',
  confirmacionContrasena: '',
  numeroTelefono: '',
  numeroCelular: '',
  // Paso 3
  direccionResidencia: '',
  codigoPostal: ''
})

// Lista de países
const paises = ref([])
const cargandoPaises = ref(false)

// Estado del modal
const mostrarModal = ref(false)

// Estados de validación para cada campo (true = válido, false = inválido, null = sin validar)
const validacionCampos = ref({
  // Paso 1
  pais: null,
  genero: null,
  primerNombre: null,
  segundoNombre: null,
  fechaNacimiento: null,
  tipoDocumento: null,
  numeroDocumento: null,
  fotoDocumentoFrente: null,
  fotoDocumentoReverso: null,
  // Paso 2
  correoElectronico: null,
  contrasena: null,
  confirmacionContrasena: null,
  numeroTelefono: null,
  numeroCelular: null,
  // Paso 3
  direccionResidencia: null,
  codigoPostal: null
})

// Cargar países desde API
const cargarPaises = async () => {
  cargandoPaises.value = true
  try {
    const response = await fetch('https://restcountries.com/v3.1/all?fields=name,translations')
    const data = await response.json()
    // Ordenar países alfabéticamente por nombre en español o inglés
    paises.value = data
      .map(country => ({
        nombre: country.translations?.spa?.common || country.name.common,
        codigo: country.name.common
      }))
      .sort((a, b) => a.nombre.localeCompare(b.nombre, 'es'))
  } catch (error) {
    console.error('Error al cargar países:', error)
    // Lista de respaldo con algunos países principales
    paises.value = [
      { nombre: 'Colombia', codigo: 'CO' },
      { nombre: 'México', codigo: 'MX' },
      { nombre: 'España', codigo: 'ES' },
      { nombre: 'Argentina', codigo: 'AR' },
      { nombre: 'Chile', codigo: 'CL' }
    ]
  } finally {
    cargandoPaises.value = false
  }
}

// Validar campo requerido
const validarCampoRequerido = (campo, nombreCampo) => {
  if (!formData.value[campo] || formData.value[campo].toString().trim() === '') {
    validacionCampos.value[nombreCampo] = false
    return false
  }
  validacionCampos.value[nombreCampo] = true
  return true
}

// Validar fecha de nacimiento (mínimo 18 años)
const validarFechaNacimiento = () => {
  if (!formData.value.fechaNacimiento) {
    errors.value.fechaNacimiento = ''
    validacionCampos.value.fechaNacimiento = null
    return
  }

  const fechaNac = new Date(formData.value.fechaNacimiento)
  const hoy = new Date()
  let edad = hoy.getFullYear() - fechaNac.getFullYear()
  const mes = hoy.getMonth() - fechaNac.getMonth()

  if (mes < 0 || (mes === 0 && hoy.getDate() < fechaNac.getDate())) {
    edad--
  }

  if (edad < 18) {
    errors.value.fechaNacimiento = 'Debe tener al menos 18 años'
    validacionCampos.value.fechaNacimiento = false
  } else {
    errors.value.fechaNacimiento = ''
    validacionCampos.value.fechaNacimiento = true
  }
}

// Prevenir entrada de caracteres no numéricos
const soloNumeros = (event) => {
  const char = String.fromCharCode(event.which)
  if (!/[0-9]/.test(char)) {
    event.preventDefault()
  }
}

// Validar número de documento (solo dígitos, mínimo 5)
const validarNumeroDocumento = () => {
  const numero = formData.value.numeroDocumento
  if (!numero) {
    errors.value.numeroDocumento = ''
    validacionCampos.value.numeroDocumento = null
    return
  }

  // Solo permitir dígitos
  if (!/^\d+$/.test(numero)) {
    formData.value.numeroDocumento = numero.replace(/\D/g, '')
  }

  if (formData.value.numeroDocumento.length > 0 && formData.value.numeroDocumento.length < 5) {
    errors.value.numeroDocumento = 'El número de documento debe tener al menos 5 dígitos'
    validacionCampos.value.numeroDocumento = false
  } else {
    errors.value.numeroDocumento = ''
    validacionCampos.value.numeroDocumento = formData.value.numeroDocumento.length >= 5
  }
}

// Manejar carga de archivo JPG
const manejarFoto = (event, tipo) => {
  const archivo = event.target.files[0]
  if (!archivo) {
    if (tipo === 'frente') {
      formData.value.fotoDocumentoFrente = null
      errors.value.fotoDocumentoFrente = ''
      validacionCampos.value.fotoDocumentoFrente = false
    } else {
      formData.value.fotoDocumentoReverso = null
      errors.value.fotoDocumentoReverso = ''
      validacionCampos.value.fotoDocumentoReverso = false
    }
    return
  }

  // Validar que sea JPG
  const esJPG = archivo.type === 'image/jpeg' || archivo.type === 'image/jpg' || archivo.name.toLowerCase().endsWith('.jpg')
  
  if (!esJPG) {
    if (tipo === 'frente') {
      errors.value.fotoDocumentoFrente = 'Solo se permiten archivos JPG'
      formData.value.fotoDocumentoFrente = null
      validacionCampos.value.fotoDocumentoFrente = false
    } else {
      errors.value.fotoDocumentoReverso = 'Solo se permiten archivos JPG'
      formData.value.fotoDocumentoReverso = null
      validacionCampos.value.fotoDocumentoReverso = false
    }
    event.target.value = ''
    return
  }

  if (tipo === 'frente') {
    formData.value.fotoDocumentoFrente = archivo
    errors.value.fotoDocumentoFrente = ''
    validacionCampos.value.fotoDocumentoFrente = true
  } else {
    formData.value.fotoDocumentoReverso = archivo
    errors.value.fotoDocumentoReverso = ''
    validacionCampos.value.fotoDocumentoReverso = true
  }
}

// Validar correo electrónico
const validarCorreo = () => {
  const correo = formData.value.correoElectronico
  if (!correo) {
    errors.value.correoElectronico = ''
    validacionCampos.value.correoElectronico = null
    return
  }

  const regexCorreo = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!regexCorreo.test(correo)) {
    errors.value.correoElectronico = 'Ingrese un correo electrónico válido'
    validacionCampos.value.correoElectronico = false
  } else {
    errors.value.correoElectronico = ''
    validacionCampos.value.correoElectronico = true
  }
}

// Validar confirmación de contraseña
const validarConfirmacionContrasena = () => {
  const contrasena = formData.value.contrasena
  const confirmacion = formData.value.confirmacionContrasena

  if (!confirmacion) {
    errors.value.confirmacionContrasena = ''
    validacionCampos.value.confirmacionContrasena = null
    return
  }

  if (contrasena !== confirmacion) {
    errors.value.confirmacionContrasena = 'Las contraseñas no coinciden'
    validacionCampos.value.confirmacionContrasena = false
  } else {
    errors.value.confirmacionContrasena = ''
    validacionCampos.value.confirmacionContrasena = contrasena !== '' && confirmacion !== ''
  }
  
  // También validar la contraseña principal
  validacionCampos.value.contrasena = contrasena !== ''
}

// Validar campos del Paso 3
const validarDireccion = () => {
  if (!formData.value.direccionResidencia || formData.value.direccionResidencia.trim() === '') {
    validacionCampos.value.direccionResidencia = false
  } else {
    validacionCampos.value.direccionResidencia = true
  }
}

const validarCodigoPostal = () => {
  if (!formData.value.codigoPostal || formData.value.codigoPostal.trim() === '') {
    validacionCampos.value.codigoPostal = false
  } else {
    validacionCampos.value.codigoPostal = true
  }
}

// Validar si un paso está completo
const validarPaso = (paso) => {
  if (paso === 1) {
    return formData.value.pais !== '' &&
           formData.value.genero !== '' &&
           formData.value.primerNombre !== '' &&
           formData.value.fechaNacimiento !== '' &&
           formData.value.tipoDocumento !== '' &&
           formData.value.numeroDocumento !== '' &&
           formData.value.numeroDocumento.length >= 5 &&
           formData.value.fotoDocumentoFrente !== null &&
           formData.value.fotoDocumentoReverso !== null &&
           validacionCampos.value.fechaNacimiento === true &&
           validacionCampos.value.numeroDocumento === true
  } else if (paso === 2) {
    return formData.value.correoElectronico !== '' &&
           formData.value.contrasena !== '' &&
           formData.value.confirmacionContrasena !== '' &&
           formData.value.numeroCelular !== '' &&
           validacionCampos.value.correoElectronico === true &&
           validacionCampos.value.confirmacionContrasena === true &&
           validacionCampos.value.numeroCelular === true
  } else if (paso === 3) {
    return formData.value.direccionResidencia !== '' &&
           formData.value.codigoPostal !== '' &&
           validacionCampos.value.direccionResidencia === true &&
           validacionCampos.value.codigoPostal === true
  }
  return false
}

const nextStep = () => {
  if (validarPaso(currentStep.value) && currentStep.value < totalSteps) {
    currentStep.value++
  }
}

const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--
  }
}

// Enviar formulario
const enviarFormulario = () => {
  if (!validarPaso(3)) {
    return
  }

  // Crear objeto con datos para imprimir (sin archivos)
  const datosEnvio = {
    // Paso 1
    pais: formData.value.pais,
    genero: formData.value.genero,
    primerNombre: formData.value.primerNombre,
    segundoNombre: formData.value.segundoNombre,
    fechaNacimiento: formData.value.fechaNacimiento,
    tipoDocumento: formData.value.tipoDocumento,
    numeroDocumento: formData.value.numeroDocumento,
    fotoDocumentoFrente: formData.value.fotoDocumentoFrente?.name || 'No seleccionado',
    fotoDocumentoReverso: formData.value.fotoDocumentoReverso?.name || 'No seleccionado',
    // Paso 2
    correoElectronico: formData.value.correoElectronico,
    contrasena: '***',
    confirmacionContrasena: '***',
    numeroTelefono: formData.value.numeroTelefono,
    numeroCelular: formData.value.numeroCelular,
    // Paso 3
    direccionResidencia: formData.value.direccionResidencia,
    codigoPostal: formData.value.codigoPostal
  }

  // Imprimir en consola
  console.log('Formulario enviado:', datosEnvio)
  console.table(datosEnvio)

  // Mostrar modal
  mostrarModal.value = true
}

onMounted(() => {
  cargarPaises()
})
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50 py-12 px-4">
    <div class="w-full max-w-2xl">
      <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
        <!-- Indicador de Pasos -->
        <div class="bg-white px-8 py-8 border-b border-gray-200">
          <div class="flex items-center justify-between">
            <!-- Paso 1 -->
            <div class="flex items-center">
              <div
                :class="[
                  'w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold transition-all duration-200 border-2',
                  currentStep >= 1
                    ? 'bg-slate-900 text-white border-slate-900'
                    : 'bg-white text-gray-400 border-gray-300'
                ]"
              >
                <span v-if="currentStep > 1">✓</span>
                <span v-else>1</span>
              </div>
              <span
                :class="[
                  'ml-3 text-sm font-medium hidden sm:block transition-colors',
                  currentStep >= 1 ? 'text-slate-900' : 'text-gray-400'
                ]"
              >
                Paso 1
              </span>
            </div>

            <!-- Línea conectora 1 -->
            <div
              :class="[
                'flex-1 h-0.5 mx-4 transition-all duration-200',
                currentStep >= 2 ? 'bg-slate-900' : 'bg-gray-200'
              ]"
            ></div>

            <!-- Paso 2 -->
            <div class="flex items-center">
              <div
                :class="[
                  'w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold transition-all duration-200 border-2',
                  currentStep >= 2
                    ? 'bg-slate-900 text-white border-slate-900'
                    : 'bg-white text-gray-400 border-gray-300'
                ]"
              >
                <span v-if="currentStep > 2">✓</span>
                <span v-else>2</span>
              </div>
              <span
                :class="[
                  'ml-3 text-sm font-medium hidden sm:block transition-colors',
                  currentStep >= 2 ? 'text-slate-900' : 'text-gray-400'
                ]"
              >
                Paso 2
              </span>
            </div>

            <!-- Línea conectora 2 -->
            <div
              :class="[
                'flex-1 h-0.5 mx-4 transition-all duration-200',
                currentStep >= 3 ? 'bg-slate-900' : 'bg-gray-200'
              ]"
            ></div>

            <!-- Paso 3 -->
            <div class="flex items-center">
              <div
                :class="[
                  'w-8 h-8 rounded-full flex items-center justify-center text-sm font-semibold transition-all duration-200 border-2',
                  currentStep >= 3
                    ? 'bg-slate-900 text-white border-slate-900'
                    : 'bg-white text-gray-400 border-gray-300'
                ]"
              >
                3
              </div>
              <span
                :class="[
                  'ml-3 text-sm font-medium hidden sm:block transition-colors',
                  currentStep >= 3 ? 'text-slate-900' : 'text-gray-400'
                ]"
              >
                Paso 3
              </span>
            </div>
          </div>
        </div>

        <!-- Contenido del Formulario -->
        <div class="p-8">
          <!-- Paso 1 -->
          <div v-if="currentStep === 1" class="space-y-5 animate-fade-in">
            <div>
              <h2 class="text-2xl font-semibold text-gray-900 mb-1">Información Personal</h2>
              <p class="text-gray-500 text-sm">Complete sus datos personales</p>
            </div>

            <form class="space-y-5" @submit.prevent>
              <!-- País -->
              <div>
                <label for="pais" class="block text-sm font-medium text-gray-700 mb-1.5">
                  País <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <select
                    id="pais"
                    v-model="formData.pais"
                    @change="validacionCampos.pais = formData.pais !== ''"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    :disabled="cargandoPaises"
                    required
                  >
                    <option value="">Seleccione un país</option>
                    <option v-for="pais in paises" :key="pais.codigo" :value="pais.nombre">
                      {{ pais.nombre }}
                    </option>
                  </select>
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.pais === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.pais === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
                <p v-if="cargandoPaises" class="text-xs text-gray-500 mt-1">Cargando países...</p>
              </div>

              <!-- Género -->
              <div>
                <label for="genero" class="block text-sm font-medium text-gray-700 mb-1.5">
                  Género <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <select
                    id="genero"
                    v-model="formData.genero"
                    @change="validacionCampos.genero = formData.genero !== ''"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    required
                  >
                    <option value="">Seleccione un género</option>
                    <option value="masculino">Masculino</option>
                    <option value="femenino">Femenino</option>
                    <option value="otro">Otro</option>
                    <option value="prefiero-no-decir">Prefiero no decir</option>
                  </select>
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.genero === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.genero === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
              </div>

              <!-- Nombres -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                <div>
                  <label for="primerNombre" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Primer Nombre <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <input
                      id="primerNombre"
                      v-model="formData.primerNombre"
                      type="text"
                      @input="validacionCampos.primerNombre = formData.primerNombre.trim() !== ''"
                      @blur="validacionCampos.primerNombre = formData.primerNombre.trim() !== '' ? true : false"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      required
                    />
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.primerNombre === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.primerNombre === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                </div>
                <div>
                  <label for="segundoNombre" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Segundo Nombre
                  </label>
                  <input
                    id="segundoNombre"
                    v-model="formData.segundoNombre"
                    type="text"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                  />
                </div>
              </div>

              <!-- Fecha de Nacimiento -->
              <div>
                <label for="fechaNacimiento" class="block text-sm font-medium text-gray-700 mb-1.5">
                  Fecha de Nacimiento <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <input
                    id="fechaNacimiento"
                    v-model="formData.fechaNacimiento"
                    type="date"
                    @change="validarFechaNacimiento"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    :max="new Date(new Date().setFullYear(new Date().getFullYear() - 18)).toISOString().split('T')[0]"
                    required
                  />
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.fechaNacimiento === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.fechaNacimiento === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
                <p v-if="errors.fechaNacimiento" class="text-xs text-red-600 mt-1">
                  {{ errors.fechaNacimiento }}
                </p>
              </div>

              <!-- Tipo y Número de Documento -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                <div>
                  <label for="tipoDocumento" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Tipo de Documento <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <select
                      id="tipoDocumento"
                      v-model="formData.tipoDocumento"
                      @change="validacionCampos.tipoDocumento = formData.tipoDocumento !== ''"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      required
                    >
                      <option value="">Seleccione un tipo</option>
                      <option value="cedula-ciudadania">Cédula de ciudadanía</option>
                      <option value="pasaporte">Pasaporte</option>
                      <option value="cedula-extranjeria">Cédula de extranjería</option>
                    </select>
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.tipoDocumento === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.tipoDocumento === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                </div>
                <div>
                  <label for="numeroDocumento" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Número de Documento <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <input
                      id="numeroDocumento"
                      v-model="formData.numeroDocumento"
                      type="text"
                      @input="validarNumeroDocumento"
                      @keypress="soloNumeros"
                      pattern="\d+"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      required
                    />
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.numeroDocumento === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.numeroDocumento === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                  <p v-if="errors.numeroDocumento" class="text-xs text-red-600 mt-1">
                    {{ errors.numeroDocumento }}
                  </p>
                </div>
              </div>

              <!-- Fotos del Documento -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                <div>
                  <label for="fotoFrente" class="flex items-center gap-2 text-sm font-medium text-gray-700 mb-1.5">
                    Foto Documento - Frente <span class="text-red-500">*</span>
                    <svg v-if="validacionCampos.fotoDocumentoFrente === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.fotoDocumentoFrente === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </label>
                  <input
                    id="fotoFrente"
                    type="file"
                    accept=".jpg,.jpeg,image/jpeg"
                    @change="manejarFoto($event, 'frente')"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm file:mr-4 file:py-1.5 file:px-3 file:rounded-md file:border-0 file:text-sm file:font-medium file:bg-gray-50 file:text-gray-700 hover:file:bg-gray-100 cursor-pointer"
                    required
                  />
                  <p v-if="errors.fotoDocumentoFrente" class="text-xs text-red-600 mt-1">
                    {{ errors.fotoDocumentoFrente }}
                  </p>
                  <p v-if="formData.fotoDocumentoFrente" class="text-xs text-gray-500 mt-1">
                    Archivo seleccionado: {{ formData.fotoDocumentoFrente.name }}
                  </p>
                  <p class="text-xs text-gray-400 mt-1">Formato: JPG</p>
                </div>
                <div>
                  <label for="fotoReverso" class="flex items-center gap-2 text-sm font-medium text-gray-700 mb-1.5">
                    Foto Documento - Reverso <span class="text-red-500">*</span>
                    <svg v-if="validacionCampos.fotoDocumentoReverso === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.fotoDocumentoReverso === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </label>
                  <input
                    id="fotoReverso"
                    type="file"
                    accept=".jpg,.jpeg,image/jpeg"
                    @change="manejarFoto($event, 'reverso')"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm file:mr-4 file:py-1.5 file:px-3 file:rounded-md file:border-0 file:text-sm file:font-medium file:bg-gray-50 file:text-gray-700 hover:file:bg-gray-100 cursor-pointer"
                    required
                  />
                  <p v-if="errors.fotoDocumentoReverso" class="text-xs text-red-600 mt-1">
                    {{ errors.fotoDocumentoReverso }}
                  </p>
                  <p v-if="formData.fotoDocumentoReverso" class="text-xs text-gray-500 mt-1">
                    Archivo seleccionado: {{ formData.fotoDocumentoReverso.name }}
                  </p>
                  <p class="text-xs text-gray-400 mt-1">Formato: JPG</p>
                </div>
              </div>
            </form>
          </div>

          <!-- Paso 2 -->
          <div v-if="currentStep === 2" class="space-y-5 animate-fade-in">
            <div>
              <h2 class="text-2xl font-semibold text-gray-900 mb-1">Datos de Contacto</h2>
              <p class="text-gray-500 text-sm">Complete su información de contacto y credenciales</p>
            </div>

            <form class="space-y-5" @submit.prevent>
              <!-- Correo electrónico -->
              <div>
                <label for="correoElectronico" class="block text-sm font-medium text-gray-700 mb-1.5">
                  Correo Electrónico <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <input
                    id="correoElectronico"
                    v-model="formData.correoElectronico"
                    type="email"
                    @blur="validarCorreo"
                    @input="validarCorreo"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    placeholder="ejemplo@correo.com"
                    required
                  />
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.correoElectronico === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.correoElectronico === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
                <p v-if="errors.correoElectronico" class="text-xs text-red-600 mt-1">
                  {{ errors.correoElectronico }}
                </p>
              </div>

              <!-- Contraseñas -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                <div>
                  <label for="contrasena" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Contraseña <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <input
                      id="contrasena"
                      v-model="formData.contrasena"
                      type="password"
                      @input="validarConfirmacionContrasena"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      required
                    />
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.contrasena === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.contrasena === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                </div>
                <div>
                  <label for="confirmacionContrasena" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Confirmación de Contraseña <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <input
                      id="confirmacionContrasena"
                      v-model="formData.confirmacionContrasena"
                      type="password"
                      @input="validarConfirmacionContrasena"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      required
                    />
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.confirmacionContrasena === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.confirmacionContrasena === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                  <p v-if="errors.confirmacionContrasena" class="text-xs text-red-600 mt-1">
                    {{ errors.confirmacionContrasena }}
                  </p>
                </div>
              </div>

              <!-- Teléfonos -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                <div>
                  <label for="numeroTelefono" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Número Teléfono
                  </label>
                  <input
                    id="numeroTelefono"
                    v-model="formData.numeroTelefono"
                    type="text"
                    @keypress="soloNumeros"
                    @input="formData.numeroTelefono = formData.numeroTelefono.replace(/\D/g, '')"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    placeholder="Ej: 1234567"
                  />
                </div>
                <div>
                  <label for="numeroCelular" class="block text-sm font-medium text-gray-700 mb-1.5">
                    Número Celular <span class="text-red-500">*</span>
                  </label>
                  <div class="relative">
                    <input
                      id="numeroCelular"
                      v-model="formData.numeroCelular"
                      type="text"
                      @keypress="soloNumeros"
                      @input="formData.numeroCelular = formData.numeroCelular.replace(/\D/g, ''); validacionCampos.numeroCelular = formData.numeroCelular !== ''"
                      @blur="validacionCampos.numeroCelular = formData.numeroCelular !== '' ? true : false"
                      class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                      placeholder="Ej: 3001234567"
                      required
                    />
                    <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                      <svg v-if="validacionCampos.numeroCelular === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                      </svg>
                      <svg v-else-if="validacionCampos.numeroCelular === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                        <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                      </svg>
                    </div>
                  </div>
                </div>
              </div>
            </form>
          </div>

          <!-- Paso 3 -->
          <div v-if="currentStep === 3" class="space-y-5 animate-fade-in">
            <div>
              <h2 class="text-2xl font-semibold text-gray-900 mb-1">Dirección</h2>
              <p class="text-gray-500 text-sm">Complete su información de dirección</p>
            </div>

            <form class="space-y-5" @submit.prevent>
              <!-- Dirección residencia -->
              <div>
                <label for="direccionResidencia" class="block text-sm font-medium text-gray-700 mb-1.5">
                  Dirección Residencia <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <input
                    id="direccionResidencia"
                    v-model="formData.direccionResidencia"
                    type="text"
                    @input="validarDireccion"
                    @blur="validarDireccion"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    required
                  />
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.direccionResidencia === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.direccionResidencia === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
              </div>

              <!-- Código postal -->
              <div>
                <label for="codigoPostal" class="block text-sm font-medium text-gray-700 mb-1.5">
                  Código Postal <span class="text-red-500">*</span>
                </label>
                <div class="relative">
                  <input
                    id="codigoPostal"
                    v-model="formData.codigoPostal"
                    type="text"
                    @input="validarCodigoPostal"
                    @blur="validarCodigoPostal"
                    class="w-full px-3 py-2 pr-10 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-1 focus:ring-slate-900 focus:border-slate-900 text-sm"
                    required
                  />
                  <div class="absolute inset-y-0 right-0 flex items-center pr-3 pointer-events-none">
                    <svg v-if="validacionCampos.codigoPostal === true" class="w-5 h-5 text-green-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
                    </svg>
                    <svg v-else-if="validacionCampos.codigoPostal === false" class="w-5 h-5 text-red-500" fill="currentColor" viewBox="0 0 20 20">
                      <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
                    </svg>
                  </div>
                </div>
              </div>
            </form>
          </div>

          <!-- Botones de Navegación -->
          <div class="flex justify-between items-center mt-10 pt-6 border-t border-gray-200">
            <button
              @click="prevStep"
              :disabled="currentStep === 1"
              :class="[
                'px-5 py-2.5 text-sm font-medium rounded-md transition-colors duration-200',
                currentStep === 1
                  ? 'text-gray-300 cursor-not-allowed'
                  : 'text-gray-700 hover:text-slate-900 hover:bg-gray-50'
              ]"
            >
              ← Anterior
            </button>

            <div class="text-xs text-gray-400">
              Paso {{ currentStep }} de {{ totalSteps }}
            </div>

            <button
              v-if="currentStep < totalSteps"
              @click="nextStep"
              :disabled="!validarPaso(currentStep)"
              :class="[
                'px-5 py-2.5 text-sm font-medium rounded-md transition-colors duration-200',
                !validarPaso(currentStep)
                  ? 'bg-gray-200 text-gray-400 cursor-not-allowed'
                  : 'bg-slate-900 text-white hover:bg-slate-800'
              ]"
            >
              Siguiente →
            </button>
            <button
              v-else
              @click="enviarFormulario"
              :disabled="!validarPaso(3)"
              :class="[
                'px-5 py-2.5 text-sm font-medium rounded-md transition-colors duration-200',
                !validarPaso(3)
                  ? 'bg-gray-200 text-gray-400 cursor-not-allowed'
                  : 'bg-slate-900 text-white hover:bg-slate-800'
              ]"
            >
              Enviar
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal de Éxito -->
    <div
      v-if="mostrarModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
      @click.self="mostrarModal = false"
    >
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full p-6 animate-fade-in">
        <div class="text-center">
          <div class="mx-auto flex items-center justify-center h-12 w-12 rounded-full bg-green-100 mb-4">
            <svg class="h-6 w-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
            </svg>
          </div>
          <h3 class="text-lg font-semibold text-gray-900 mb-2">¡Formulario Enviado!</h3>
          <p class="text-sm text-gray-500 mb-6">
            Su formulario ha sido enviado con éxito. Los datos han sido registrados correctamente.
          </p>
          <button
            @click="mostrarModal = false"
            class="w-full px-4 py-2 bg-slate-900 text-white rounded-md hover:bg-slate-800 transition-colors duration-200 text-sm font-medium"
          >
            Cerrar
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fade-in 0.3s ease-out;
}
</style>
