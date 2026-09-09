<template>

  <q-layout view="lHh Lpr lFf">

    <q-header class="header">

      <q-toolbar class="barra-superior">

        <div class="marca-header">

          <q-avatar
            color="white"
            text-color="primary"
            size="42px"
          >
            <q-icon name="build" size="23px" />
          </q-avatar>

        <q-toolbar-title>

          <div class="titulo">
            Servicio Técnico
          </div>

          <div class="subtitulo">
            Gestión de celulares y tablets
          </div>

        </q-toolbar-title>

        </div>

        <q-btn
          label="Nuevo servicio"
          icon="add"
          color="white"
          text-color="primary"
          unelevated
          class="boton-nuevo"
          @click="abrirNuevoServicio"
        />

      </q-toolbar>

    </q-header>

    <q-page-container>

      <q-page class="pagina">

        <q-card class="buscador q-mb-lg">

          <q-card-section>

            <q-input
              v-model="busqueda"
              outlined
              rounded
              clearable
              label="Buscar cliente o equipo"
              placeholder="Ej: Danna, Samsung A15, iPhone 12..."
            >

            </q-input>

          </q-card-section>

        </q-card>

        <div class="row q-col-gutter-md q-mb-lg">

          <div class="col-12 col-sm-4">

            <q-card class="estadistica">

              <q-card-section>

                <div class="estadistica-contenido">

                  <div class="color-circle primary"></div>

                  <div>

                    <div class="texto-estadistica">
                      Total de servicios
                    </div>

                    <div class="numero">
                      {{ servicios.length }}
                    </div>

                  </div>

                </div>

              </q-card-section>

            </q-card>

          </div>

          <div class="col-12 col-sm-4">

            <q-card class="estadistica">

              <q-card-section>

                <div class="estadistica-contenido">

                  <div class="color-circle warning"></div>

                  <div>

                    <div class="texto-estadistica">
                      En reparación
                    </div>

                    <div class="numero">
                      {{ contarEstado('en reparación') }}
                    </div>

                  </div>

                </div>

              </q-card-section>

            </q-card>

          </div>

          <div class="col-12 col-sm-4">

            <q-card class="estadistica">

              <q-card-section>

                <div class="estadistica-contenido">

                  <div class="color-circle negative"></div>

                  <div>

                    <div class="texto-estadistica">
                      Pagos pendientes
                    </div>

                    <div class="numero">
                      {{ contarPago('pendiente') }}
                    </div>

                  </div>

                </div>

              </q-card-section>

            </q-card>

          </div>

        </div>

        <div
          v-if="servicios.length === 0"
          class="mensaje-vacio"
        >

          <q-icon
            name="phone_android"
            size="80px"
          />

          <h4>
            No hay servicios registrados
          </h4>

          <p>
            Comienza registrando el primer equipo del taller.
          </p>

          <q-btn
            label="Registrar servicio"
            icon="add"
            color="primary"
            unelevated
            @click="abrirNuevoServicio"
          />

        </div>

        <div
          v-else-if="serviciosFiltrados.length > 0"
          class="row q-col-gutter-md"
        >

          <div
            v-for="resultado in serviciosFiltrados"
            :key="resultado.indice"
            class="col-12 col-md-6 col-lg-4"
          >

            <q-card
              class="tarjeta"
              :class="{
                'pago-pendiente': resultado.servicio.estadoPago === 'pendiente',
                'pago-abono': resultado.servicio.estadoPago === 'abono'
              }"
            >

              <q-card-section>

                <div class="row items-center justify-between">

                  <div class="row items-center">

                    <q-avatar
                      color="primary"
                      text-color="white"
                    >

                      <q-icon name="phone_android" />

                    </q-avatar>


                    <div class="q-ml-md">

                      <div class="nombre-cliente">
                        {{ resultado.servicio.cliente }}
                      </div>

                      <div class="equipo">
                        {{ resultado.servicio.marca }} {{ resultado.servicio.modelo || resultado.servicio.equipo }}
                      </div>

                    </div>

                  </div>

                  <div>

                    <q-icon
                      v-if="resultado.servicio.estadoEquipo === 'recibido'"
                      name="inventory_2"
                      color="blue"
                      size="30px"
                    />

                    <q-icon
                      v-else-if="resultado.servicio.estadoEquipo === 'en reparación'"
                      name="build"
                      color="orange"
                      size="30px"
                    />

                    <q-icon
                      v-else-if="resultado.servicio.estadoEquipo === 'listo para entregar'"
                      name="check_circle"
                      color="green"
                      size="30px"
                    />

                    <q-icon
                      v-else-if="resultado.servicio.estadoEquipo === 'entregado'"
                      name="done_all"
                      color="grey"
                      size="30px"
                    />

                  </div>

                </div>

              </q-card-section>

              <q-separator />

              <q-card-section>

                <div class="dato">

                  <q-icon name="build" />

                  <span>
                    {{ textoReparaciones(resultado.servicio) }}
                  </span>

                </div>

                <div class="dato">

                  <q-icon name="person" />

                  <span>
                    Técnico: {{ resultado.servicio.tecnico }}
                  </span>

                </div>

                <div class="dato">

                  <q-icon name="schedule" />

                  <span>
                    {{ resultado.servicio.fecha }}
                  </span>

                </div>

                <div class="dato">

                  <q-icon name="attach_money" />

                  <span>
                    ${{ Number(resultado.servicio.precio).toFixed(2) }}
                  </span>

                </div>

                <div class="dato">

                  <q-icon name="payments" />

                  <span>
                    {{ resultado.servicio.metodoPago }}
                  </span>

                </div>

                <div class="q-mt-md">

                  <div class="titulo-pequeno">
                    Estado del equipo
                  </div>


                  <q-badge
                    v-if="resultado.servicio.estadoEquipo === 'recibido'"
                    color="blue"
                    class="q-mt-xs"
                  >

                    Equipo recibido

                  </q-badge>


                  <q-badge
                    v-else-if="resultado.servicio.estadoEquipo === 'en reparación'"
                    color="orange"
                    class="q-mt-xs"
                  >

                    En reparación

                  </q-badge>


                  <q-badge
                    v-else-if="resultado.servicio.estadoEquipo === 'listo para entregar'"
                    color="green"
                    class="q-mt-xs"
                  >

                    Listo para entregar

                  </q-badge>


                  <q-badge
                    v-else
                    color="grey"
                    class="q-mt-xs"
                  >

                    Entregado

                  </q-badge>

                </div>

                <div class="q-mt-md">

                  <div class="titulo-pequeno">
                    Estado del pago
                  </div>


                  <q-badge
                    v-if="resultado.servicio.estadoPago === 'pagado'"
                    color="positive"
                    class="q-mt-xs"
                  >

                    ✓ Pagado

                  </q-badge>


                  <q-badge
                    v-else-if="resultado.servicio.estadoPago === 'abono'"
                    color="warning"
                    class="q-mt-xs"
                  >

                    Abono: ${{ Number(resultado.servicio.montoAbono || 0).toFixed(2) }}

                  </q-badge>


                  <q-badge
                    v-else
                    color="negative"
                    class="q-mt-xs"
                  >

                    ✕ Pago pendiente

                  </q-badge>

                </div>

                <div
                  v-if="resultado.servicio.calificacion > 0"
                  class="q-mt-md"
                >

                  <div class="titulo-pequeno">
                    Calificación del cliente
                  </div>

                  <q-rating
                    v-model="resultado.servicio.calificacion"
                    readonly
                    size="25px"
                    icon="star_border"
                    icon-selected="star"
                  />

                </div>

                <div
                  v-if="resultado.servicio.observaciones"
                  class="observaciones q-mt-md"
                >

                  <strong>
                    Observaciones
                  </strong>

                  <div class="q-mt-xs">
                    {{ resultado.servicio.observaciones }}
                  </div>

                </div>

              </q-card-section>


              <q-card-actions align="right">

                <q-btn
                  v-if="resultado.servicio.estadoEquipo === 'entregado' && resultado.servicio.estadoPago === 'pagado' && !resultado.servicio.calificacion"
                  flat
                  icon="star"
                  label="Calificar"
                  color="amber-8"
                  @click="abrirCalificacion(resultado.servicio)"
                />

                <q-btn
                  v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                  flat
                  icon="edit"
                  label="Editar"
                  color="primary"
                  @click="editarServicio(resultado.indice)"
                />

                <q-btn
                  v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                  flat
                  icon="delete"
                  label="Eliminar"
                  color="negative"
                  @click="confirmarEliminar(resultado.indice)"
                />

              </q-card-actions>

            </q-card>

          </div>

        </div>

        <div
          v-else
          class="mensaje-vacio"
        >

          <q-icon
            name="search_off"
            size="70px"
          />

          <h4>
            No encontramos resultados
          </h4>

          <p>
            No hay clientes o equipos que coincidan con:
          </p>

          <strong>
            "{{ busqueda }}"
          </strong>

        </div>

        <q-dialog v-model="mostrarModal">

          <q-card class="modal">

            <q-card-section class="row items-center">

              <div class="text-h6">

                {{ editando ? 'Editar servicio' : 'Nuevo servicio' }}

              </div>

              <q-space />

              <q-btn
                icon="close"
                flat
                round
                dense
                @click="cerrarModal"
              />

            </q-card-section>


            <q-separator />

            <q-form @submit="guardarServicio">

              <q-card-section>

                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  :rules="[reglaNombre]"
                  class="q-mb-md"
                />

                <q-select
                  v-model="formulario.marca"
                  label="Marca *"
                  outlined
                  :options="marcas"
                  @update:model-value="cambiarMarca"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-select
                  v-if="formulario.marca !== 'Otra'"
                  v-model="formulario.modelo"
                  label="Modelo *"
                  outlined
                  :options="modelosDisponibles"
                  :disable="!formulario.marca"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-input
                  v-else
                  v-model="formulario.modelo"
                  label="Modelo *"
                  placeholder="Escribe el modelo"
                  outlined
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-select
                  v-model="formulario.reparacion"
                  label="Tipo de reparación *"
                  outlined
                  :options="reparaciones"
                  multiple
                  use-chips
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-input
                  v-if="formulario.reparacion.includes('Otros')"
                  v-model="formulario.reparacionOtro"
                  label="Especifica el tipo de servicio *"
                  placeholder="Ej: Reparación de cámara"
                  outlined
                  :rules="[reglaOtro]"
                  class="q-mb-md"
                />

                <q-select
                  v-model="formulario.tecnico"
                  label="Técnico *"
                  outlined
                  :options="tecnicos"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-input
                  v-model="formulario.fecha"
                  label="Fecha y hora de recepción *"
                  type="datetime-local"
                  outlined
                  readonly
                  class="q-mb-md"
                />

                <q-input
                  v-model="formulario.precio"
                  label="Precio cobrado *"
                  type="number"
                  prefix="$"
                  min="0"
                  step="0.01"
                  outlined
                  :rules="[reglaPrecio]"
                  class="q-mb-md"
                />

                <q-select
                  v-model="formulario.metodoPago"
                  label="Método de pago *"
                  outlined
                  :options="metodosPago"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-select
                  v-model="formulario.estadoPago"
                  label="Estado del pago *"
                  outlined
                  :options="estadosPago"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <div
                  v-if="mostrarErrorEntrega"
                  class="text-negative text-caption q-mb-md"
                >
                  Para entregar el equipo, el estado del pago debe ser pagado.
                </div>

                <q-select
                  v-model="formulario.estadoEquipo"
                  label="Estado del equipo *"
                  outlined
                  :options="estadosEquipo"
                  :disable="!editando"
                  :rules="[reglaObligatoria]"
                  class="q-mb-md"
                />

                <q-input
                  v-if="formulario.estadoPago === 'abono'"
                  v-model="formulario.montoAbono"
                  label="Valor del abono *"
                  type="number"
                  prefix="$"
                  min="0.01"
                  :max="formulario.precio"
                  step="0.01"
                  outlined
                  :rules="[reglaAbono]"
                  class="q-mb-md"
                />

                <q-input
                  v-model="formulario.observaciones"
                  label="Observaciones"
                  outlined
                  type="textarea"
                  rows="3"
                  placeholder="Ej: Pantalla partida en la esquina superior..."
                />

              </q-card-section>

              <q-card-actions align="right">

                <q-btn
                  label="Cancelar"
                  flat
                  color="grey"
                  @click="cerrarModal"
                />

                <q-btn
                  type="submit"
                  :label="editando ? 'Guardar cambios' : 'Registrar servicio'"
                  color="primary"
                  unelevated
                />

              </q-card-actions>

            </q-form>

          </q-card>

        </q-dialog>

        <q-dialog v-model="mostrarCalificacion">

          <q-card class="calificacion-modal">

            <q-card-section>

              <div class="text-h6">
                Calificación del cliente
              </div>

              <div class="text-grey-7 q-mt-sm">
                {{ servicioCalificar?.cliente }} - {{ servicioCalificar?.marca }} {{ servicioCalificar?.modelo }}
              </div>

            </q-card-section>

            <q-card-section class="text-center">

              <q-rating
                v-model="calificacionPendiente"
                size="42px"
                color="amber-8"
                icon="star_border"
                icon-selected="star"
              />

              <div
                v-if="mostrarErrorCalificacion"
                class="text-negative text-caption q-mt-sm"
              >
                Selecciona una calificación para continuar.
              </div>

            </q-card-section>

            <q-card-actions align="right">

              <q-btn
                label="Cancelar"
                flat
                @click="cerrarCalificacion"
              />

              <q-btn
                label="Guardar calificación"
                color="primary"
                unelevated
                @click="guardarCalificacion"
              />

            </q-card-actions>

          </q-card>

        </q-dialog>

        <q-dialog v-model="mostrarConfirmacion">

          <q-card class="confirmacion">


            <q-card-section class="row items-center">

              <q-avatar
                color="negative"
                text-color="white"
              >

                <q-icon name="delete" />

              </q-avatar>

              <div class="q-ml-md">

                <div class="text-h6">
                  Eliminar servicio
                </div>

                <div class="text-grey-7">
                  Esta acción no se puede deshacer.
                </div>

              </div>

            </q-card-section>


            <q-card-section>

              ¿Está seguro de que desea eliminar este servicio?

            </q-card-section>


            <q-card-actions align="right">

              <q-btn
                label="Cancelar"
                flat
                @click="cerrarConfirmacion"
              />

              <q-btn
                label="Eliminar"
                color="negative"
                unelevated
                @click="eliminarServicio"
              />

            </q-card-actions>

          </q-card>

        </q-dialog>

      </q-page>

    </q-page-container>

  </q-layout>

</template>

<script setup>

import { computed, ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage(
  'servicios-tecnicos',
  []
)

const mostrarModal = ref(false)

const mostrarConfirmacion = ref(false)

const editando = ref(false)

const busqueda = ref('')

const indiceEditar = ref(-1)

const indiceEliminar = ref(-1)

const mostrarErrorCalificacion = ref(false)

const mostrarErrorEntrega = ref(false)

const mostrarCalificacion = ref(false)

const servicioCalificar = ref(null)

const calificacionPendiente = ref(0)

function fechaActual() {

  const ahora = new Date()

  const fecha = new Date(
    ahora.getTime() - ahora.getTimezoneOffset() * 60000
  )

  return fecha.toISOString().slice(0, 16)

}

function crearFormulario() {

  return {

    cliente: '',

    marca: '',

    modelo: '',

    reparacion: [],

    reparacionOtro: '',

    tecnico: '',

    fecha: fechaActual(),

    precio: '',

    montoAbono: '',

    metodoPago: '',

    estadoPago: '',

    estadoEquipo: 'recibido',

    calificacion: 0,

    observaciones: ''

  }

}

const formulario = ref(crearFormulario())

const reparaciones = [

  'Cambio de pantalla',

  'Cambio de batería',

  'Cambio de pin de carga',

  'Liberación',

  'Mantenimiento de software',

  'Cambio de flex',

  'Otros'

]

const marcas = [

  'Apple',

  'Samsung',

  'Xiaomi',

  'Motorola',

  'Huawei',

  'Oppo',

  'Honor',

  'Nokia',

  'Otra'

]

const modelosPorMarca = {

  Apple: [
    'iPhone 11',
    'iPhone 12',
    'iPhone 13',
    'iPhone 14',
    'iPhone 15',
    'iPad'
  ],

  Samsung: [
    'Galaxy A15',
    'Galaxy A25',
    'Galaxy A54',
    'Galaxy S23',
    'Galaxy S24',
    'Galaxy Note 20'
  ],

  Xiaomi: [
    'Redmi Note 12',
    'Redmi Note 13',
    'Redmi 12',
    'Poco X5',
    'Poco X6'
  ],

  Motorola: [
    'Moto G32',
    'Moto G53',
    'Moto G84',
    'Edge 40'
  ],

  Huawei: [
    'P30 Lite',
    'P40 Lite',
    'Nova 9',
    'Mate 40 Pro'
  ],

  Oppo: [
    'A57',
    'A78',
    'Reno 8',
    'Reno 10'
  ],

  Honor: [
    'X7',
    'X8',
    'X9',
    'Magic 5 Lite'
  ],

  Nokia: [
    'C21',
    'G22',
    'G42',
    'X30'
  ],

  Otra: []

}

const modelosDisponibles = computed(() => {

  const modelos = modelosPorMarca[formulario.value.marca] || []

  if (
    formulario.value.modelo &&
    !modelos.includes(formulario.value.modelo)
  ) {

    return [formulario.value.modelo, ...modelos]

  }

  return modelos

})


const tecnicos = [

  'Don Efraín',

  'Andres',

  'Santiago'

]


const metodosPago = [

  'Efectivo',

  'Transferencia',

  'Tarjeta'

]


const estadosPago = [

  'pagado',

  'pendiente',

  'abono'

]


const estadosEquipo = [

  'recibido',

  'en reparación',

  'listo para entregar',

  'entregado'

]

function reglaObligatoria(valor) {

  if (
    valor === null ||
    valor === undefined ||
    (typeof valor === 'string' && !valor.trim()) ||
    (Array.isArray(valor) && valor.length === 0)
  ) {

    return 'Este campo es obligatorio'

  }

  return true

}

function reglaNombre(valor) {

  if (!valor || !String(valor).trim()) {

    return 'El nombre del cliente es obligatorio'

  }

  if (!/[A-Za-zÁÉÍÓÚáéíóúÑñ]/.test(String(valor))) {

    return 'Escribe un nombre válido'

  }

  return true

}

function reglaPrecio(valor) {

  if (
    valor === null ||
    valor === undefined ||
    valor === ''
  ) {

    return 'El precio es obligatorio'

  }


  if (Number(valor) < 0) {

    return 'El precio no puede ser negativo'

  }


  return true

}

function reglaOtro(valor) {

  if (!formulario.value.reparacion.includes('Otros')) {

    return true

  }

  if (!valor || !String(valor).trim()) {

    return 'Especifica el tipo de servicio'

  }

  return true

}

function textoReparaciones(servicio) {

  const reparacionesSeleccionadas = Array.isArray(servicio.reparacion)
    ? servicio.reparacion
    : [servicio.reparacion].filter(Boolean)

  const reparacionesTexto = reparacionesSeleccionadas
    .filter((reparacion) => reparacion !== 'Otros')

  if (reparacionesSeleccionadas.includes('Otros')) {

    reparacionesTexto.push(
      servicio.reparacionOtro
        ? `Otros: ${servicio.reparacionOtro}`
        : 'Otros'
    )

  }

  return reparacionesTexto.join(', ')

}

function reglaAbono(valor) {

  if (formulario.value.estadoPago !== 'abono') {

    return true

  }

  if (valor === null || valor === undefined || valor === '') {

    return 'El valor del abono es obligatorio'

  }

  if (Number(valor) <= 0 || Number(valor) > Number(formulario.value.precio)) {

    return 'El abono debe ser mayor que cero y no superar el precio'

  }

  return true

}

function buscarServicios() {

  let resultados = []

  const texto = busqueda.value
    .toLowerCase()
    .trim()


  for (
    let i = 0;
    i < servicios.value.length;
    i++
  ) {

    const cliente =
      servicios.value[i].cliente
        .toLowerCase()


    const equipo = [
      servicios.value[i].marca,
      servicios.value[i].modelo,
      servicios.value[i].equipo
    ]
      .filter(Boolean)
      .join(' ')
      .toLowerCase()


    if (

      cliente.includes(texto) ||

      equipo.includes(texto)

    ) {

      resultados.push({

        servicio: servicios.value[i],

        indice: i

      })

    }

  }


  return resultados

}

const serviciosFiltrados = computed(() => buscarServicios())

function limpiarFormulario() {

  formulario.value = crearFormulario()

}

function cambiarMarca() {

  formulario.value.modelo = ''

}

function abrirNuevoServicio() {

  limpiarFormulario()

  editando.value = false

  indiceEditar.value = -1

  mostrarErrorEntrega.value = false

  mostrarModal.value = true

}

function cerrarModal() {

  mostrarModal.value = false

  limpiarFormulario()

  mostrarErrorEntrega.value = false

}

function abrirCalificacion(servicio) {

  servicioCalificar.value = servicio

  calificacionPendiente.value = servicio.calificacion || 0

  mostrarErrorCalificacion.value = false

  mostrarCalificacion.value = true

}

function cerrarCalificacion() {

  mostrarCalificacion.value = false

  servicioCalificar.value = null

  calificacionPendiente.value = 0

  mostrarErrorCalificacion.value = false

}

function guardarCalificacion() {

  if (!servicioCalificar.value || calificacionPendiente.value === 0) {

    mostrarErrorCalificacion.value = true

    return

  }

  servicioCalificar.value.calificacion = calificacionPendiente.value

  cerrarCalificacion()

}

function guardarServicio() {

  mostrarErrorCalificacion.value = false

  mostrarErrorEntrega.value = false

  if (
    formulario.value.estadoEquipo === 'entregado' &&
    formulario.value.estadoPago !== 'pagado'
  ) {

    mostrarErrorEntrega.value = true

    return

  }

  const servicio = {

    ...formulario.value,

    cliente: String(formulario.value.cliente || '').trim(),

    marca: String(formulario.value.marca || '').trim(),

    modelo: String(formulario.value.modelo || '').trim(),

    observaciones: String(formulario.value.observaciones || '').trim(),

    reparacion: Array.isArray(formulario.value.reparacion)
      ? formulario.value.reparacion
      : [formulario.value.reparacion].filter(Boolean),

    reparacionOtro: formulario.value.reparacion.includes('Otros')
      ? String(formulario.value.reparacionOtro || '').trim()
      : '',

    precio: Number(formulario.value.precio),

    montoAbono: formulario.value.estadoPago === 'abono'
      ? Number(formulario.value.montoAbono)
      : 0,

    calificacion: Number(formulario.value.calificacion) || 0

  }


  if (editando.value) {


    if (indiceEditar.value < 0 || indiceEditar.value >= servicios.value.length) {

      return

    }

    servicios.value[indiceEditar.value] = servicio


  } else {


    servicios.value.push(servicio)

  }


  mostrarModal.value = false

  limpiarFormulario()

  editando.value = false

  indiceEditar.value = -1

  if (servicio.estadoEquipo === 'entregado') {

    abrirCalificacion(servicio)

  }

}

function editarServicio(index) {

  if (
    index < 0 ||
    index >= servicios.value.length ||
    servicios.value[index].estadoEquipo === 'entregado'
  ) {

    return

  }

  const servicio = servicios.value[index]

  formulario.value = {

    ...crearFormulario(),

    ...servicio,

    marca: servicio.marca || '',

    modelo: servicio.modelo || servicio.equipo || '',

    reparacion: Array.isArray(servicio.reparacion)
      ? servicio.reparacion
      : [servicio.reparacion].filter(Boolean),

    reparacionOtro: servicio.reparacionOtro || '',

    montoAbono: servicio.montoAbono || ''

  }


  indiceEditar.value = index

  editando.value = true

  mostrarModal.value = true

}

function confirmarEliminar(index) {

  if (
    index < 0 ||
    index >= servicios.value.length ||
    servicios.value[index].estadoEquipo === 'entregado'
  ) {

    return

  }

  indiceEliminar.value = index

  mostrarConfirmacion.value = true

}

function cerrarConfirmacion() {

  mostrarConfirmacion.value = false

  indiceEliminar.value = -1

}

function eliminarServicio() {

  if (
    indiceEliminar.value >= 0 &&
    indiceEliminar.value < servicios.value.length &&
    servicios.value[indiceEliminar.value].estadoEquipo !== 'entregado'
  ) {

    servicios.value.splice(
      indiceEliminar.value,
      1
    )

  }


  cerrarConfirmacion()

}

function contarEstado(estado) {

  let cantidad = 0


  for (
    let i = 0;
    i < servicios.value.length;
    i++
  ) {

    if (
      servicios.value[i].estadoEquipo === estado
    ) {

      cantidad++

    }

  }


  return cantidad

}

function contarPago(estado) {

  let cantidad = 0


  for (
    let i = 0;
    i < servicios.value.length;
    i++
  ) {

    if (
      servicios.value[i].estadoPago === estado
    ) {

      cantidad++

    }

  }


  return cantidad

}

</script>

<style scoped>

.pagina {

  background:
    linear-gradient(180deg, #f8fbfc 0%, #eef4f5 100%);

  font-family: "Avenir Next", "Trebuchet MS", sans-serif;

  min-height: 100vh;

  padding: 30px clamp(16px, 4vw, 56px) 48px;

}

.header {

  background: linear-gradient(115deg, #173f46 0%, #245c62 100%);

  box-shadow: 0 4px 18px rgba(18, 52, 58, 0.24);

}

.barra-superior {

  min-height: 76px;

  padding: 10px clamp(16px, 4vw, 56px);

}

.marca-header {

  display: flex;

  align-items: center;

  flex: 1;

  min-width: 0;

  gap: 13px;

}

.boton-nuevo {

  border-radius: 10px;

  padding: 0 18px;

  font-weight: 700;

  text-transform: none;

  letter-spacing: 0;

  margin-left: 18px;

}

.titulo {

  font-size: 21px;

  font-weight: bold;

  letter-spacing: 0.1px;

}

.subtitulo {

  font-size: 14px;

  opacity: 0.9;

  letter-spacing: 0.2px;

}

.buscador {

  border: 1px solid #e0eaec;

  border-radius: 14px;

  background: white;

  box-shadow: 0 8px 24px rgba(42, 76, 82, 0.07) !important;

}

.buscador :deep(.q-field__control) {

  border-radius: 10px;

  background: #fbfdfd;


}

.estadistica {

  border: 1px solid #e0eaec;

  border-radius: 14px;

  height: 100%;

  background: rgba(255, 255, 255, 0.94);

  box-shadow: 0 8px 22px rgba(42, 76, 82, 0.06) !important;

}

.estadistica-contenido {

  display: flex;

  align-items: center;

  gap: 15px;

}

.texto-estadistica {

  color: #6b7280;

  font-size: 15px;

}

.numero {

  font-size: 30px;

  font-weight: bold;

  color: #164b58;

}

.color-circle {
  width: 50px;
  height: 50px;
  border-radius: 14px;
  flex-shrink: 0;
}

.color-circle.primary {
  background: #007bff;

  border: 1px solid #0066d6;
}

.color-circle.warning {
  background: #fed843;

  border: 1px solid #e5bd26;
}

.color-circle.negative {
  background: #ec1000;

  border: 1px solid #c90e00;
}

.tarjeta {

  border: 1px solid #e0eaec;

  border-radius: 14px;

  height: 100%;

  overflow: hidden;

  background: rgba(255, 255, 255, 0.96);

  box-shadow: 0 9px 24px rgba(42, 76, 82, 0.07) !important;

}


.pago-pendiente {

  border-left: 6px solid #c62828;

}


.pago-abono {

  border-left: 6px solid #f2a900;

}

.nombre-cliente {

  font-size: 17px;

  font-weight: bold;

  color: #20383c;

}


.equipo {

  color: #6b7280;

  font-size: 15px;

}

.dato {

  display: flex;

  align-items: center;

  gap: 10px;

  margin-bottom: 11px;

  color: #555;

  font-size: 15px;

}


.dato .q-icon {

  color: #29626D;

}

.titulo-pequeno {

  color: #6b7280;

  font-size: 14px;

  font-weight: 600;

}

.observaciones {

  background: #f1f3f4;

  border-radius: 10px;

  padding: 12px;

  color: #555;

}

.mensaje-vacio {

  text-align: center;

  padding: 70px 20px;

  color: #777;

}


.mensaje-vacio h4 {

  margin-bottom: 8px;

  color: #444;

}


.mensaje-vacio p {

  margin-top: 5px;

}

:deep(.q-card) {
  font-family: "Avenir Next", "Trebuchet MS", sans-serif;
}

:deep(.q-dialog__inner) {
  padding: 24px;
}

:deep(.q-btn) {

  border-radius: 9px;

  text-transform: none;

  letter-spacing: 0;

  transition: none !important;

}

:deep(.q-card),
:deep(.q-field__control),
:deep(.q-focus-helper) {

  transition: none !important;

}

:deep(.q-field--outlined .q-field__control:before) {

  border-color: #d7e4e6;

}

:deep(.q-field--outlined.q-field--focused .q-field__control:after) {

  border-color: #347b7b;

}

.modal {

  width: 600px;

  max-width: 95vw;

  border-radius: 14px;
  
  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;

}

.confirmacion {

  width: 450px;

  max-width: 95vw;

  border-radius: 14px;

  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;

}

.calificacion-modal {

  width: 410px;

  max-width: 95vw;

  border-radius: 14px;

  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;

}

@media (max-width: 600px) {

  .pagina {

    padding: 12px;

  }

  .titulo {

    font-size: 17px;

  }

  .subtitulo {

    display: none;

  }

  .header .q-btn {

    font-size: 11px;

  }

  .tarjeta {

    border-radius: 15px;

  }

}

</style>