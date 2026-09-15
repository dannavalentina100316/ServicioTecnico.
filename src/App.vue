<template>
  <q-layout view="lHh Lpr lFf">
    <!-- Barra Superior / Header Original -->
    <q-header class="header" elevated>
      <q-toolbar class="barra-superior">
        <div class="marca-header cursor-pointer" @click="resetearFiltros">
          <q-avatar
            color="white"
            text-color="primary"
            size="44px"
            class="shadow-2"
          >
            <q-icon name="build" size="24px" />
          </q-avatar>
          <q-toolbar-title>
            <div class="titulo">
              Servicio Técnico
            </div>
            <div class="subtitulo">
              Taller de Celulares y Tablets - Don Efraín
            </div>
          </q-toolbar-title>
        </div>

        <q-btn
          label="Nuevo servicio"
          icon="add"
          color="white"
          text-color="primary"
          unelevated
          class="boton-nuevo shadow-1"
          @click="abrirNuevoServicio"
        />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="pagina">

        <!-- Tarjetas de Estadísticas / Navegación rápida interactiva -->
        <div class="row q-col-gutter-md q-mb-md">
          <!-- Tarjeta 1: Total de servicios -->
          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer transition-card"
              :class="{ 'tarjeta-activa': filtroEstado === 'todos' && filtroPago === 'todos' }"
              @click="seleccionarFiltroRapido('todos')"
            >
              <q-card-section class="tarjeta-cabecera">
                <div class="estadistica-contenido">
                  <div class="color-circle primary">
                    <q-icon name="inventory_2" size="26px" color="white" />
                  </div>
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

          <!-- Tarjeta 2: En reparación -->
          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer transition-card"
              :class="{ 'tarjeta-activa': filtroEstado === 'en reparación' }"
              @click="seleccionarFiltroRapido('en reparación')"
            >
              <q-card-section class="tarjeta-cabecera">
                <div class="estadistica-contenido">
                  <div class="color-circle warning">
                    <q-icon name="build" size="26px" color="white" />
                  </div>
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

          <!-- Tarjeta 3: Listos para entregar -->
          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer transition-card"
              :class="{ 'tarjeta-activa': filtroEstado === 'listo para entregar' }"
              @click="seleccionarFiltroRapido('listo para entregar')"
            >
              <q-card-section class="tarjeta-cabecera">
                <div class="estadistica-contenido">
                  <div class="color-circle success">
                    <q-icon name="check_circle" size="26px" color="white" />
                  </div>
                  <div>
                    <div class="texto-estadistica">
                      Listos p/ entrega
                    </div>
                    <div class="numero">
                      {{ contarEstado('listo para entregar') }}
                    </div>
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>

          <!-- Tarjeta 4: Pagos pendientes / Abonos (Cuánto se fio) -->
          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer transition-card"
              :class="{ 'tarjeta-activa': filtroPago === 'con-saldo' }"
              @click="seleccionarFiltroRapido('con-saldo')"
            >
              <q-card-section class="tarjeta-cabecera">
                <div class="estadistica-contenido">
                  <div class="color-circle negative">
                    <q-icon name="payments" size="26px" color="white" />
                  </div>
                  <div>
                    <div class="texto-estadistica">
                      Saldo por cobrar
                    </div>
                    <div class="numero numero-dinero">
                      {{ formatoPesosCorto(calcularTotalPendienteCobro()) }}
                    </div>
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- Barra de Navegación, Búsqueda y Filtros Mejorada -->
        <q-card class="buscador q-mb-lg">
          <q-card-section class="q-pa-md">
            <!-- Fila superior: Input de búsqueda + Selector de orden -->
            <div class="row q-col-gutter-sm items-center">
              <div class="col-12 col-md-8">
                <q-input
                  v-model="busqueda"
                  @update:model-value="buscarServicios"
                  @clear="buscarServicios"
                  outlined
                  rounded
                  clearable
                  dense
                  label="Buscar cliente, equipo, técnico o falla"
                  placeholder="Ej: Danna, Samsung A15, iPhone, cambio de pantalla..."
                  class="input-busqueda"
                >
                  <template v-slot:prepend>
                    <q-icon name="search" color="primary" />
                  </template>
                </q-input>
              </div>

              <div class="col-12 col-md-4">
                <q-select
                  v-model="criterioOrden"
                  @update:model-value="buscarServicios"
                  outlined
                  rounded
                  dense
                  label="Ordenar por"
                  :options="opcionesOrden"
                  emit-value
                  map-options
                  class="select-orden"
                >
                  <template v-slot:prepend>
                    <q-icon name="sort" color="primary" />
                  </template>
                </q-select>
              </div>
            </div>

            <!-- Fila inferior: Pestañas / Chips de Filtrado Rápido -->
            <div class="filtros-navegacion q-mt-sm">
              <span class="etiqueta-filtrar text-weight-bold q-mr-sm">
                Filtrar:
              </span>
              <div class="chips-container">
                <q-btn
                  :color="filtroEstado === 'todos' && filtroPago === 'todos' ? 'primary' : 'grey-3'"
                  :text-color="filtroEstado === 'todos' && filtroPago === 'todos' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="Todos"
                  @click="cambiarFiltroEstado('todos')"
                />

                <q-btn
                  :color="filtroEstado === 'recibido' ? 'blue-8' : 'grey-3'"
                  :text-color="filtroEstado === 'recibido' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="Recibidos"
                  icon="inventory_2"
                  @click="cambiarFiltroEstado('recibido')"
                />

                <q-btn
                  :color="filtroEstado === 'en reparación' ? 'orange-9' : 'grey-3'"
                  :text-color="filtroEstado === 'en reparación' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="En reparación"
                  icon="build"
                  @click="cambiarFiltroEstado('en reparación')"
                />

                <q-btn
                  :color="filtroEstado === 'listo para entregar' ? 'positive' : 'grey-3'"
                  :text-color="filtroEstado === 'listo para entregar' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="Listos"
                  icon="check_circle"
                  @click="cambiarFiltroEstado('listo para entregar')"
                />

                <q-btn
                  :color="filtroEstado === 'entregado' ? 'grey-8' : 'grey-3'"
                  :text-color="filtroEstado === 'entregado' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="Entregados"
                  icon="done_all"
                  @click="cambiarFiltroEstado('entregado')"
                />

                <q-btn
                  :color="filtroPago === 'con-saldo' ? 'negative' : 'grey-3'"
                  :text-color="filtroPago === 'con-saldo' ? 'white' : 'dark'"
                  unelevated
                  rounded
                  dense
                  size="sm"
                  class="filtro-chip"
                  label="Con saldo pendiente (Fiado)"
                  icon="money_off"
                  @click="cambiarFiltroPago('con-saldo')"
                />

                <q-btn
                  v-if="filtroEstado !== 'todos' || filtroPago !== 'todos' || busqueda"
                  flat
                  rounded
                  dense
                  size="sm"
                  color="primary"
                  label="Restablecer filtros"
                  icon="refresh"
                  class="q-ml-sm"
                  @click="resetearFiltros"
                />
              </div>
            </div>
          </q-card-section>
        </q-card>

        <!-- Mensaje cuando la lista total de servicios está vacía -->
        <div
          v-if="servicios.length === 0"
          class="mensaje-vacio"
        >
          <q-icon
            name="phone_android"
            size="80px"
            color="grey-5"
          />
          <h4>
            No hay servicios registrados
          </h4>
          <p>
            Comienza registrando el primer equipo que Don Efraín recibe en el taller.
          </p>
          <q-btn
            label="Registrar servicio"
            icon="add"
            color="primary"
            unelevated
            class="q-mt-md"
            @click="abrirNuevoServicio"
          />
        </div>

        <!-- Listado de Tarjetas de Servicios -->
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
              class="tarjeta transition-card"
              :class="{
                'pago-pendiente': resultado.servicio.estadoPago === 'pendiente',
                'pago-abono': resultado.servicio.estadoPago === 'abono',
                'equipo-entregado': resultado.servicio.estadoEquipo === 'entregado',
                'equipo-listo': resultado.servicio.estadoEquipo === 'listo para entregar'
              }"
            >
              <!-- Cabecera de la tarjeta -->
              <q-card-section class="tarjeta-cabecera">
                <div class="row items-center justify-between no-wrap">
                  <div class="row items-center no-wrap ellipsis">
                    <q-avatar
                      color="primary"
                      text-color="white"
                      size="40px"
                      class="q-mr-md"
                    >
                      <q-icon :name="iconoPorMarca(resultado.servicio.marca)" />
                    </q-avatar>

                    <div class="ellipsis">
                      <div class="nombre-cliente ellipsis">
                        {{ resultado.servicio.cliente }}
                      </div>
                      <div class="equipo ellipsis">
                        {{ resultado.servicio.marca }} {{ resultado.servicio.modelo || resultado.servicio.equipo }}
                      </div>
                    </div>
                  </div>

                  <!-- Ícono destacado de estado del equipo -->
                  <div class="q-ml-sm text-right">
                    <q-badge
                      v-if="resultado.servicio.estadoEquipo === 'recibido'"
                      color="blue-1"
                      text-color="blue-9"
                      class="q-pa-xs badge-estado"
                    >
                      <q-icon name="inventory_2" size="20px" class="q-mr-xs" />
                      Recibido
                    </q-badge>

                    <q-badge
                      v-else-if="resultado.servicio.estadoEquipo === 'en reparación'"
                      color="orange-1"
                      text-color="orange-9"
                      class="q-pa-xs badge-estado"
                    >
                      <q-icon name="build" size="20px" class="q-mr-xs" />
                      En reparación
                    </q-badge>

                    <q-badge
                      v-else-if="resultado.servicio.estadoEquipo === 'listo para entregar'"
                      color="green-1"
                      text-color="green-9"
                      class="q-pa-xs badge-estado badge-pulse"
                    >
                      <q-icon name="check_circle" size="20px" class="q-mr-xs" />
                      ¡Listo!
                    </q-badge>

                    <q-badge
                      v-else-if="resultado.servicio.estadoEquipo === 'entregado'"
                      color="grey-3"
                      text-color="grey-8"
                      class="q-pa-xs badge-estado"
                    >
                      <q-icon name="done_all" size="20px" class="q-mr-xs" />
                      Entregado
                    </q-badge>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <!-- Contenido principal de la tarjeta -->
              <q-card-section class="tarjeta-contenido">
                <div class="datos-grid">
                  <!-- Reparación requerida -->
                  <div class="dato">
                    <q-icon name="build" />
                    <span>
                      {{ textoReparaciones(resultado.servicio) }}
                    </span>
                  </div>

                  <!-- Técnico a cargo -->
                  <div class="dato">
                    <q-icon name="person" />
                    <span>
                      Técnico: <strong>{{ resultado.servicio.tecnico }}</strong>
                    </span>
                  </div>

                  <!-- Fecha de ingreso -->
                  <div class="dato">
                    <q-icon name="schedule" />
                    <span>
                      {{ formatoFecha(resultado.servicio.fecha) }}
                    </span>
                  </div>

                  <!-- Precio pactado -->
                  <div class="dato">
                    <q-icon name="attach_money" />
                    <span>
                      Total: <strong>{{ formatoPesos(resultado.servicio.precio) }}</strong>
                    </span>
                  </div>

                  <!-- Método de pago -->
                  <div class="dato">
                    <q-icon name="payments" />
                    <span>
                      Pago: {{ resultado.servicio.metodoPago }}
                    </span>
                  </div>

                  <!-- Teléfono / Contacto (si fue registrado) -->
                  <div class="dato" v-if="resultado.servicio.telefono">
                    <q-icon name="phone" />
                    <span>
                      Tel: {{ resultado.servicio.telefono }}
                    </span>
                  </div>
                </div>

                <!-- Bloques de Estado y Deuda / Fiado -->
                <div class="estados-grid q-mt-md">
                  <!-- Estado del Equipo con botón de avance rápido de estado para Don Efraín -->
                  <div class="estado-bloque">
                    <div class="titulo-pequeno">
                      Estado del equipo
                    </div>

                    <div class="q-mt-xs row items-center no-wrap">
                      <q-badge
                        v-if="resultado.servicio.estadoEquipo === 'recibido'"
                        color="blue"
                        class="q-py-xs q-px-sm"
                      >
                        Recibido
                      </q-badge>
                      <q-badge
                        v-else-if="resultado.servicio.estadoEquipo === 'en reparación'"
                        color="orange"
                        class="q-py-xs q-px-sm"
                      >
                        En reparación
                      </q-badge>
                      <q-badge
                        v-else-if="resultado.servicio.estadoEquipo === 'listo para entregar'"
                        color="green"
                        class="q-py-xs q-px-sm"
                      >
                        Listo para entregar
                      </q-badge>
                      <q-badge
                        v-else
                        color="grey"
                        class="q-py-xs q-px-sm"
                      >
                        Entregado
                      </q-badge>

                      <!-- Botón de avance rápido de estado (Gran ayuda a la navegabilidad) -->
                      <q-btn
                        v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                        flat
                        dense
                        round
                        size="xs"
                        color="primary"
                        icon="arrow_forward"
                        class="q-ml-sm"
                        title="Avanzar estado del equipo"
                        @click="avanzarEstadoRapido(resultado.indice)"
                      >
                        <q-tooltip>
                          Avanzar a: {{ siguienteEstadoNombre(resultado.servicio.estadoEquipo) }}
                        </q-tooltip>
                      </q-btn>
                    </div>
                  </div>

                  <!-- Estado del Pago y Cuánto se le fio al cliente -->
                  <div class="estado-bloque">
                    <div class="titulo-pequeno">
                      Estado del pago
                    </div>

                    <div class="q-mt-xs">
                      <q-badge
                        v-if="resultado.servicio.estadoPago === 'pagado'"
                        color="positive"
                        class="q-py-xs q-px-sm"
                      >
                        ✓ Pagado completo
                      </q-badge>

                      <div v-else-if="resultado.servicio.estadoPago === 'abono'">
                        <q-badge
                          color="warning"
                          text-color="dark"
                          class="q-py-xs q-px-sm text-weight-bold"
                        >
                          Abonó: {{ formatoPesos(resultado.servicio.montoAbono) }}
                        </q-badge>
                        <div class="deuda-texto q-mt-xs">
                          Resta pagar: <strong>{{ formatoPesos(calcularDeuda(resultado.servicio)) }}</strong>
                        </div>
                      </div>

                      <div v-else>
                        <q-badge
                          color="negative"
                          class="q-py-xs q-px-sm text-weight-bold"
                        >
                          ✕ Pago pendiente
                        </q-badge>
                        <div class="deuda-texto q-mt-xs">
                          Debe: <strong>{{ formatoPesos(resultado.servicio.precio) }}</strong>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- Calificación del cliente (Estrellas) -->
                <div
                  v-if="resultado.servicio.calificacion > 0"
                  class="estado-bloque q-mt-sm"
                >
                  <div class="titulo-pequeno">
                    Calificación del cliente
                  </div>
                  <div class="row items-center q-mt-xs">
                    <q-rating
                      v-model="resultado.servicio.calificacion"
                      readonly
                      size="22px"
                      color="amber-8"
                      icon="star_border"
                      icon-selected="star"
                    />
                    <span class="text-caption text-grey-8 q-ml-sm">
                      ({{ resultado.servicio.calificacion }} de 5 estrellas)
                    </span>
                  </div>
                </div>

                <!-- Observaciones técnicas / Estado de ingreso -->
                <div
                  v-if="resultado.servicio.observaciones"
                  class="observaciones estado-bloque q-mt-sm"
                >
                  <strong>Observaciones:</strong>
                  <div class="q-mt-xs text-caption">
                    {{ resultado.servicio.observaciones }}
                  </div>
                </div>
              </q-card-section>

              <!-- Botones de Acción -->
              <q-card-actions align="between" class="tarjeta-acciones">
                <div>
                  <!-- Botón de WhatsApp rápido si tiene teléfono y está listo -->
                  <q-btn
                    v-if="resultado.servicio.telefono && resultado.servicio.estadoEquipo === 'listo para entregar'"
                    flat
                    round
                    dense
                    color="positive"
                    icon="chat"
                    title="Notificar cliente por WhatsApp"
                    @click="abrirWhatsApp(resultado.servicio)"
                  >
                    <q-tooltip>Avisar al cliente por WhatsApp</q-tooltip>
                  </q-btn>
                </div>

                <div class="row items-center q-gutter-xs">
                  <!-- Botón de Calificar (si está entregado y pagado pero no calificado) -->
                  <q-btn
                    v-if="resultado.servicio.estadoEquipo === 'entregado' && resultado.servicio.estadoPago === 'pagado' && !resultado.servicio.calificacion"
                    flat
                    icon="star"
                    label="Calificar"
                    color="amber-9"
                    size="sm"
                    @click="abrirCalificacion(resultado.servicio)"
                  />

                  <!-- Botón de Editar -->
                  <q-btn
                    v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                    flat
                    icon="edit"
                    label="Editar"
                    color="primary"
                    size="sm"
                    @click="editarServicio(resultado.indice)"
                  />

                  <!-- Botón de Eliminar -->
                  <q-btn
                    v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                    flat
                    icon="delete"
                    label="Eliminar"
                    color="negative"
                    size="sm"
                    @click="confirmarEliminar(resultado.indice)"
                  />
                </div>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <!-- Mensaje cuando los filtros o búsqueda no arrojan resultados -->
        <div
          v-else
          class="mensaje-vacio"
        >
          <q-icon
            name="search_off"
            size="70px"
            color="grey-6"
          />
          <h4>
            No encontramos resultados
          </h4>
          <p v-if="busqueda">
            No hay clientes o equipos que coincidan con la búsqueda:
            <strong class="text-primary">"{{ busqueda }}"</strong>
          </p>
          <p v-else>
            No hay servicios en la categoría seleccionada.
          </p>
          <q-btn
            label="Ver todos los servicios"
            icon="clear_all"
            color="primary"
            outline
            class="q-mt-sm"
            @click="resetearFiltros"
          />
        </div>

        <!-- Botón flotante para Don Efraín en dispositivos móviles (Navegabilidad rápida) -->
        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn
            fab
            icon="add"
            color="primary"
            text-color="white"
            class="shadow-5"
            @click="abrirNuevoServicio"
          >
            <q-tooltip anchor="top middle" self="bottom middle">
              Registrar nuevo servicio
            </q-tooltip>
          </q-btn>
        </q-page-sticky>

        <!-- Modal: Registrar / Editar Servicio -->
        <q-dialog v-model="mostrarModal" persistent>
          <q-card class="modal">
            <q-card-section class="row items-center modal-header">
              <div class="row items-center">
                <q-icon :name="editando ? 'edit' : 'add_circle'" size="26px" color="primary" class="q-mr-sm" />
                <div class="text-h6 text-weight-bold">
                  {{ editando ? 'Editar servicio técnico' : 'Nuevo servicio técnico' }}
                </div>
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
              <q-card-section class="modal-cuerpo q-pa-md">
                <!-- Nombre del cliente -->
                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  dense
                  :rules="[reglaNombre]"
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="person" />
                  </template>
                </q-input>

                <!-- Teléfono (Opcional pero muy útil para el taller) -->
                <q-input
                  v-model="formulario.telefono"
                  label="Teléfono / Celular (opcional)"
                  placeholder="Ej: 3101234567"
                  outlined
                  dense
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="phone" />
                  </template>
                </q-input>

                <!-- Marca y Modelo en dos columnas -->
                <div class="row q-col-gutter-sm q-mb-sm">
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.marca"
                      label="Marca *"
                      outlined
                      dense
                      :options="marcas"
                      @update:model-value="cambiarMarca"
                      :rules="[reglaObligatoria]"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-if="formulario.marca !== 'Otra'"
                      v-model="formulario.modelo"
                      label="Modelo *"
                      outlined
                      dense
                      :options="modelosDisponibles"
                      :disable="!formulario.marca"
                      :rules="[reglaObligatoria]"
                    />
                    <q-input
                      v-else
                      v-model="formulario.modelo"
                      label="Modelo *"
                      placeholder="Escribe el modelo"
                      outlined
                      dense
                      :rules="[reglaObligatoria]"
                    />
                  </div>
                </div>

                <!-- Tipo de Reparación -->
                <q-select
                  v-model="formulario.reparacion"
                  label="Tipo de reparación *"
                  outlined
                  dense
                  :options="reparaciones"
                  multiple
                  use-chips
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="build" />
                  </template>
                </q-select>

                <!-- Campo Otro servicio -->
                <q-input
                  v-if="formulario.reparacion.includes('Otros')"
                  v-model="formulario.reparacionOtro"
                  label="Especifica el otro tipo de servicio *"
                  placeholder="Ej: Reparación de cámara frontal"
                  outlined
                  dense
                  :rules="[reglaOtro]"
                  class="q-mb-sm"
                />

                <!-- Técnico a cargo -->
                <q-select
                  v-model="formulario.tecnico"
                  label="Técnico que atendió *"
                  outlined
                  dense
                  :options="tecnicos"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="badge" />
                  </template>
                </q-select>

                <!-- Fecha y hora de recepción -->
                <q-input
                  :model-value="formatoFecha(formulario.fecha)"
                  label="Fecha y hora de recepción *"
                  outlined
                  dense
                  readonly
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="schedule" />
                  </template>
                </q-input>

                <!-- Precios y Métodos de pago -->
                <div class="row q-col-gutter-sm q-mb-sm">
                  <div class="col-12 col-sm-6">
                    <q-input
                      v-model="formulario.precio"
                      label="Precio cobrado *"
                      type="text"
                      inputmode="numeric"
                      prefix="$"
                      hint="Pesos colombianos"
                      @update:model-value="formatearPrecio('precio', $event)"
                      outlined
                      dense
                      :rules="[reglaPrecio]"
                    >
                      <template v-slot:prepend>
                        <q-icon name="attach_money" />
                      </template>
                    </q-input>
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.metodoPago"
                      label="Método de pago *"
                      outlined
                      dense
                      :options="metodosPago"
                      :rules="[reglaObligatoria]"
                    >
                      <template v-slot:prepend>
                        <q-icon name="payments" />
                      </template>
                    </q-select>
                  </div>
                </div>

                <!-- Estado del pago -->
                <q-select
                  v-model="formulario.estadoPago"
                  label="Estado del pago *"
                  outlined
                  dense
                  :options="estadosPago"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                />

                <!-- Valor de Abono condicional -->
                <q-input
                  v-if="formulario.estadoPago === 'abono'"
                  v-model="formulario.montoAbono"
                  label="Valor del abono inicial *"
                  type="text"
                  inputmode="numeric"
                  prefix="$"
                  hint="Pesos colombianos"
                  @update:model-value="formatearPrecio('montoAbono', $event)"
                  outlined
                  dense
                  :rules="[reglaAbono]"
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="price_check" />
                  </template>
                </q-input>

                <!-- Estado del equipo -->
                <q-select
                  v-model="formulario.estadoEquipo"
                  label="Estado del equipo *"
                  outlined
                  dense
                  :options="estadosEquipo"
                  :disable="!editando"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                >
                  <template v-slot:prepend>
                    <q-icon name="devices" />
                  </template>
                </q-select>

                <!-- Alerta si se intenta entregar sin estar pagado -->
                <div
                  v-if="mostrarErrorEntrega"
                  class="q-pa-sm bg-red-1 text-negative rounded-borders q-mb-sm text-caption row items-center"
                >
                  <q-icon name="warning" size="18px" class="q-mr-xs" />
                  <span>Para entregar el equipo al cliente, el estado del pago debe ser <strong>pagado</strong>.</span>
                </div>

                <!-- Observaciones -->
                <q-input
                  v-model="formulario.observaciones"
                  label="Observaciones (opcional)"
                  outlined
                  dense
                  type="textarea"
                  rows="2"
                  placeholder="Ej: Pantalla partida en la esquina superior, cliente pide respaldar fotos..."
                />
              </q-card-section>

              <q-separator />

              <q-card-actions align="right" class="q-pa-md">
                <q-btn
                  label="Cancelar"
                  flat
                  color="grey-7"
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

        <!-- Modal: Calificación del Cliente -->
        <q-dialog v-model="mostrarCalificacion">
          <q-card class="calificacion-modal">
            <q-card-section class="bg-primary text-white row items-center justify-between">
              <div class="text-h6">
                Calificación del cliente
              </div>
              <q-btn icon="close" flat round dense @click="cerrarCalificacion" />
            </q-card-section>

            <q-card-section class="q-pa-md">
              <div class="text-subtitle1 text-weight-bold">
                {{ servicioCalificar?.cliente }}
              </div>
              <div class="text-grey-7">
                {{ servicioCalificar?.marca }} {{ servicioCalificar?.modelo }}
              </div>
              <p class="text-caption text-grey-8 q-mt-sm">
                Don Efraín, pídale al cliente que califique el servicio de reparación recibido:
              </p>

              <div class="text-center q-py-md">
                <q-rating
                  v-model="calificacionPendiente"
                  size="44px"
                  color="amber-8"
                  icon="star_border"
                  icon-selected="star"
                />
                <div
                  v-if="mostrarErrorCalificacion"
                  class="text-negative text-caption q-mt-sm text-weight-bold"
                >
                  Selecciona al menos una estrella para guardar la calificación.
                </div>
              </div>
            </q-card-section>

            <q-separator />

            <q-card-actions align="right" class="q-pa-md">
              <q-btn
                label="Omitir"
                flat
                color="grey-7"
                @click="cerrarCalificacion"
              />
              <q-btn
                label="Guardar calificación"
                color="primary"
                unelevated
                icon="check"
                @click="guardarCalificacion"
              />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- Modal: Confirmar Eliminación -->
        <q-dialog v-model="mostrarConfirmacion">
          <q-card class="confirmacion">
            <q-card-section class="row items-center">
              <q-avatar
                color="negative"
                text-color="white"
                size="44px"
              >
                <q-icon name="delete" />
              </q-avatar>
              <div class="q-ml-md">
                <div class="text-h6 text-weight-bold">
                  Eliminar servicio
                </div>
                <div class="text-grey-7 text-caption">
                  Esta acción no se puede deshacer.
                </div>
              </div>
            </q-card-section>

            <q-card-section class="q-pt-none">
              ¿Está seguro de que desea eliminar este registro de reparación del taller?
            </q-card-section>

            <q-separator />

            <q-card-actions align="right" class="q-pa-md">
              <q-btn
                label="Cancelar"
                flat
                color="grey-7"
                @click="cerrarConfirmacion"
              />
              <q-btn
                label="Eliminar registro"
                color="negative"
                unelevated
                icon="delete_forever"
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
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Datos iniciales de muestra basados en el taller de Don Efraín
const datosEjemplo = [
  {
    cliente: 'Carlos Mendoza',
    telefono: '3124567890',
    marca: 'Apple',
    modelo: 'iPhone 12',
    reparacion: ['Cambio de pantalla'],
    reparacionOtro: '',
    tecnico: 'Don Efraín',
    fecha: '2026-09-14T09:30',
    precio: 280000,
    montoAbono: 100000,
    metodoPago: 'Transferencia',
    estadoPago: 'abono',
    estadoEquipo: 'en reparación',
    calificacion: 0,
    observaciones: 'Pantalla con líneas verdes y táctil fallando. Cliente pide respaldar fotos.'
  },
  {
    cliente: 'María Fernanda Ruiz',
    telefono: '3157891234',
    marca: 'Samsung',
    modelo: 'Galaxy A15',
    reparacion: ['Cambio de batería'],
    reparacionOtro: '',
    tecnico: 'Andres',
    fecha: '2026-09-14T11:15',
    precio: 95000,
    montoAbono: 0,
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
    estadoEquipo: 'recibido',
    calificacion: 0,
    observaciones: 'Batería inflada, no retiene carga más de 20 minutos.'
  },
  {
    cliente: 'Julián Castro',
    telefono: '3006549870',
    marca: 'Xiaomi',
    modelo: 'Redmi Note 12',
    reparacion: ['Cambio de pin de carga'],
    reparacionOtro: '',
    tecnico: 'Santiago',
    fecha: '2026-09-13T16:00',
    precio: 65000,
    montoAbono: 0,
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
    estadoEquipo: 'listo para entregar',
    calificacion: 0,
    observaciones: 'Pin sulfatado. Se reemplazó y carga perfectamente a 33W.'
  },
  {
    cliente: 'Laura Gómez',
    telefono: '3189998877',
    marca: 'Motorola',
    modelo: 'Moto G53',
    reparacion: ['Mantenimiento de software', 'Liberación'],
    reparacionOtro: '',
    tecnico: 'Don Efraín',
    fecha: '2026-09-12T10:00',
    precio: 80000,
    montoAbono: 0,
    metodoPago: 'Tarjeta',
    estadoPago: 'pagado',
    estadoEquipo: 'entregado',
    calificacion: 5,
    observaciones: 'Se eliminó cuenta olvidada y se actualizó a la última versión.'
  }
]

// Persistencia con useLocalStorage según restricción
const servicios = useLocalStorage('servicios-tecnicos', datosEjemplo)

// Estados de interfaz reactivos usando únicamente ref()
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

// Estados de Navegabilidad y Filtrado
const filtroEstado = ref('todos') // 'todos', 'recibido', 'en reparación', 'listo para entregar', 'entregado'
const filtroPago = ref('todos') // 'todos', 'con-saldo', 'pagado'
const criterioOrden = ref('recientes')

const opcionesOrden = [
  { label: 'Más recientes primero', value: 'recientes' },
  { label: 'Más antiguos primero', value: 'antiguos' },
  { label: 'Mayor precio', value: 'precio-desc' },
  { label: 'Menor precio', value: 'precio-asc' },
  { label: 'Nombre del cliente (A-Z)', value: 'cliente-asc' }
]

function fechaActual() {
  const ahora = new Date()
  const fecha = new Date(ahora.getTime() - ahora.getTimezoneOffset() * 60000)
  return fecha.toISOString().slice(0, 16)
}

function crearFormulario() {
  return {
    cliente: '',
    telefono: '',
    marca: '',
    modelo: '',
    reparacion: [],
    reparacionOtro: '',
    tecnico: '',
    fecha: fechaActual(),
    precio: '',
    montoAbono: '',
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
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
  Apple: ['iPhone 11', 'iPhone 12', 'iPhone 13', 'iPhone 14', 'iPhone 15', 'iPad'],
  Samsung: ['Galaxy A15', 'Galaxy A25', 'Galaxy A54', 'Galaxy S23', 'Galaxy S24', 'Galaxy Note 20'],
  Xiaomi: ['Redmi Note 12', 'Redmi Note 13', 'Redmi 12', 'Poco X5', 'Poco X6'],
  Motorola: ['Moto G32', 'Moto G53', 'Moto G84', 'Edge 40'],
  Huawei: ['P30 Lite', 'P40 Lite', 'Nova 9', 'Mate 40 Pro'],
  Oppo: ['A57', 'A78', 'Reno 8', 'Reno 10'],
  Honor: ['X7', 'X8', 'X9', 'Magic 5 Lite'],
  Nokia: ['C21', 'G22', 'G42', 'X30'],
  Otra: []
}

const modelosDisponibles = ref([])

function actualizarModelos() {
  const modelos = modelosPorMarca[formulario.value.marca] || []
  if (formulario.value.modelo && !modelos.includes(formulario.value.modelo)) {
    modelosDisponibles.value = [formulario.value.modelo, ...modelos]
    return
  }
  modelosDisponibles.value = modelos
}

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

// Validaciones usando reglas de Quasar
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
  if (valor === null || valor === undefined || valor === '') {
    return 'El precio es obligatorio'
  }
  if (numeroSinFormato(valor) <= 0) {
    return 'El precio debe ser mayor a 0'
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

function reglaAbono(valor) {
  if (formulario.value.estadoPago !== 'abono') {
    return true
  }
  if (valor === null || valor === undefined || valor === '') {
    return 'El valor del abono es obligatorio'
  }
  const abonoNum = numeroSinFormato(valor)
  const precioNum = numeroSinFormato(formulario.value.precio)
  if (abonoNum <= 0) {
    return 'El abono debe ser mayor a cero'
  }
  if (abonoNum >= precioNum) {
    return 'El abono debe ser menor que el precio total (si pagó todo, elija "pagado")'
  }
  return true
}

function numeroSinFormato(valor) {
  return Number(String(valor || '').replace(/[^0-9]/g, '')) || 0
}

function formatoPesos(valor) {
  return `$ ${numeroSinFormato(valor).toLocaleString('es-CO')}`
}

function formatoPesosCorto(valor) {
  const num = numeroSinFormato(valor)
  if (num >= 1000000) {
    return `$ ${(num / 1000000).toFixed(1)}M`
  }
  if (num >= 1000) {
    return `$ ${(num / 1000).toFixed(0)}k`
  }
  return `$ ${num}`
}

function formatearPrecio(campo, valor) {
  const digitos = String(valor || '').replace(/[^0-9]/g, '')
  formulario.value[campo] = digitos ? Number(digitos).toLocaleString('es-CO') : ''
}

function formatoFecha(valor) {
  if (!valor) return ''
  const [fecha, hora = ''] = String(valor).replace(' ', 'T').split('T')
  const [anio, mes, dia] = fecha.split('-')
  if (!anio || !mes || !dia) return valor
  return `${dia}/${mes}/${anio}${hora ? ` - ${hora.slice(0, 5)}` : ''}`
}

function textoReparaciones(servicio) {
  const reparacionesSeleccionadas = Array.isArray(servicio.reparacion)
    ? servicio.reparacion
    : [servicio.reparacion].filter(Boolean)
  const reparacionesTexto = reparacionesSeleccionadas.filter((r) => r !== 'Otros')
  if (reparacionesSeleccionadas.includes('Otros')) {
    reparacionesTexto.push(servicio.reparacionOtro ? `Otros: ${servicio.reparacionOtro}` : 'Otros')
  }
  return reparacionesTexto.join(', ')
}

function calcularDeuda(servicio) {
  const precio = numeroSinFormato(servicio.precio)
  if (servicio.estadoPago === 'pagado') return 0
  if (servicio.estadoPago === 'abono') {
    const abono = numeroSinFormato(servicio.montoAbono)
    return Math.max(0, precio - abono)
  }
  return precio // pendiente completo
}

function calcularTotalPendienteCobro() {
  let total = 0
  for (let i = 0; i < servicios.value.length; i++) {
    total += calcularDeuda(servicios.value[i])
  }
  return total
}

function contarConSaldoPendiente() {
  let cuenta = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estadoPago === 'pendiente' || servicios.value[i].estadoPago === 'abono') {
      cuenta++
    }
  }
  return cuenta
}

function contarEstado(estado) {
  let cantidad = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estadoEquipo === estado) {
      cantidad++
    }
  }
  return cantidad
}

function contarPago(estado) {
  let cantidad = 0
  for (let i = 0; i < servicios.value.length; i++) {
    if (servicios.value[i].estadoPago === estado) {
      cantidad++
    }
  }
  return cantidad
}

function iconoPorMarca(marca) {
  const m = String(marca || '').toLowerCase()
  if (m === 'apple') return 'phone_iphone'
  if (m === 'tablet' || m === 'ipad') return 'tablet_android'
  return 'phone_android'
}

// Búsqueda, Filtrado y Ordenamiento usando funciones estándar y ref
const serviciosFiltrados = ref([])

function buscarServicios() {
  let resultados = []
  const texto = String(busqueda.value || '').toLowerCase().trim()

  for (let i = 0; i < servicios.value.length; i++) {
    const serv = servicios.value[i]

    // Filtro por Estado de Equipo
    if (filtroEstado.value !== 'todos' && serv.estadoEquipo !== filtroEstado.value) {
      continue
    }

    // Filtro por Estado de Pago
    if (filtroPago.value === 'con-saldo') {
      if (serv.estadoPago !== 'pendiente' && serv.estadoPago !== 'abono') {
        continue
      }
    } else if (filtroPago.value === 'pagado') {
      if (serv.estadoPago !== 'pagado') {
        continue
      }
    }

    // Búsqueda de texto enriquecida (cliente, equipo, marca, modelo, técnico, fallas)
    if (texto) {
      const cliente = String(serv.cliente || '').toLowerCase()
      const equipo = [serv.marca, serv.modelo, serv.equipo].filter(Boolean).join(' ').toLowerCase()
      const tecnico = String(serv.tecnico || '').toLowerCase()
      const reparacionesStr = Array.isArray(serv.reparacion) ? serv.reparacion.join(' ').toLowerCase() : ''
      const obs = String(serv.observaciones || '').toLowerCase()

      const coincide =
        cliente.includes(texto) ||
        equipo.includes(texto) ||
        tecnico.includes(texto) ||
        reparacionesStr.includes(texto) ||
        obs.includes(texto)

      if (!coincide) {
        continue
      }
    }

    resultados.push({
      servicio: serv,
      indice: i
    })
  }

  // Ordenamiento
  if (criterioOrden.value === 'recientes') {
    resultados.sort((a, b) => new Date(b.servicio.fecha) - new Date(a.servicio.fecha))
  } else if (criterioOrden.value === 'antiguos') {
    resultados.sort((a, b) => new Date(a.servicio.fecha) - new Date(b.servicio.fecha))
  } else if (criterioOrden.value === 'precio-desc') {
    resultados.sort((a, b) => numeroSinFormato(b.servicio.precio) - numeroSinFormato(a.servicio.precio))
  } else if (criterioOrden.value === 'precio-asc') {
    resultados.sort((a, b) => numeroSinFormato(a.servicio.precio) - numeroSinFormato(b.servicio.precio))
  } else if (criterioOrden.value === 'cliente-asc') {
    resultados.sort((a, b) => String(a.servicio.cliente).localeCompare(String(b.servicio.cliente)))
  }

  serviciosFiltrados.value = resultados
}

// Navegabilidad con filtros rápidos
function cambiarFiltroEstado(estado) {
  filtroEstado.value = estado
  filtroPago.value = 'todos'
  buscarServicios()
}

function cambiarFiltroPago(pago) {
  filtroPago.value = pago
  filtroEstado.value = 'todos'
  buscarServicios()
}

function seleccionarFiltroRapido(tipo) {
  if (tipo === 'todos') {
    resetearFiltros()
  } else if (tipo === 'en reparación' || tipo === 'listo para entregar') {
    cambiarFiltroEstado(tipo)
  } else if (tipo === 'con-saldo') {
    cambiarFiltroPago('con-saldo')
  }
}

function resetearFiltros() {
  busqueda.value = ''
  filtroEstado.value = 'todos'
  filtroPago.value = 'todos'
  criterioOrden.value = 'recientes'
  buscarServicios()
}

// Avance rápido de estado directamente desde la tarjeta (Mejora clave de navegabilidad)
function siguienteEstadoNombre(estadoActual) {
  if (estadoActual === 'recibido') return 'En reparación'
  if (estadoActual === 'en reparación') return 'Listo para entregar'
  if (estadoActual === 'listo para entregar') return 'Entregado'
  return ''
}

function avanzarEstadoRapido(indice) {
  if (indice < 0 || indice >= servicios.value.length) return
  const serv = servicios.value[indice]

  if (serv.estadoEquipo === 'recibido') {
    serv.estadoEquipo = 'en reparación'
  } else if (serv.estadoEquipo === 'en reparación') {
    serv.estadoEquipo = 'listo para entregar'
  } else if (serv.estadoEquipo === 'listo para entregar') {
    if (serv.estadoPago !== 'pagado') {
      // Don Efraín debe cobrar antes de entregar
      editarServicio(indice)
      mostrarErrorEntrega.value = true
      return
    }
    serv.estadoEquipo = 'entregado'
    abrirCalificacion(serv)
  }
  buscarServicios()
}

// WhatsApp directo al cliente para avisarle que su equipo está listo
function abrirWhatsApp(servicio) {
  if (!servicio.telefono) return
  const telefonoLimpio = String(servicio.telefono).replace(/[^0-9]/g, '')
  const mensaje = encodeURIComponent(
    `Hola ${servicio.cliente}, te escribimos del taller de Don Efraín. Te informamos que tu equipo (${servicio.marca} ${servicio.modelo || ''}) ya está listo para entregar. Saldo pendiente: ${formatoPesos(calcularDeuda(servicio))}. ¡Te esperamos!`
  )
  window.open(`https://wa.me/57${telefonoLimpio}?text=${mensaje}`, '_blank')
}

// Operaciones CRUD
function limpiarFormulario() {
  formulario.value = crearFormulario()
  actualizarModelos()
}

function cambiarMarca() {
  formulario.value.modelo = ''
  actualizarModelos()
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
  buscarServicios()
  cerrarCalificacion()
}

function guardarServicio() {
  mostrarErrorCalificacion.value = false
  mostrarErrorEntrega.value = false

  if (formulario.value.estadoEquipo === 'entregado' && formulario.value.estadoPago !== 'pagado') {
    mostrarErrorEntrega.value = true
    return
  }

  const servicio = {
    ...formulario.value,
    cliente: String(formulario.value.cliente || '').trim(),
    telefono: String(formulario.value.telefono || '').trim(),
    marca: String(formulario.value.marca || '').trim(),
    modelo: String(formulario.value.modelo || '').trim(),
    observaciones: String(formulario.value.observaciones || '').trim(),
    reparacion: Array.isArray(formulario.value.reparacion)
      ? formulario.value.reparacion
      : [formulario.value.reparacion].filter(Boolean),
    reparacionOtro: formulario.value.reparacion.includes('Otros')
      ? String(formulario.value.reparacionOtro || '').trim()
      : '',
    precio: numeroSinFormato(formulario.value.precio),
    montoAbono: formulario.value.estadoPago === 'abono' ? numeroSinFormato(formulario.value.montoAbono) : 0,
    calificacion: Number(formulario.value.calificacion) || 0
  }

  if (editando.value) {
    if (indiceEditar.value < 0 || indiceEditar.value >= servicios.value.length) return
    servicios.value[indiceEditar.value] = servicio
  } else {
    servicios.value.unshift(servicio) // Agregar al principio para visibilidad inmediata
  }

  buscarServicios()

  mostrarModal.value = false
  limpiarFormulario()
  const estabaEditando = editando.value
  editando.value = false
  indiceEditar.value = -1

  if (servicio.estadoEquipo === 'entregado' && !servicio.calificacion) {
    abrirCalificacion(servicio)
  }
}

function editarServicio(index) {
  if (index < 0 || index >= servicios.value.length || servicios.value[index].estadoEquipo === 'entregado') {
    return
  }
  const servicio = servicios.value[index]
  formulario.value = {
    ...crearFormulario(),
    ...servicio,
    telefono: servicio.telefono || '',
    marca: servicio.marca || '',
    modelo: servicio.modelo || servicio.equipo || '',
    precio: servicio.precio ? Number(servicio.precio).toLocaleString('es-CO') : '',
    montoAbono: servicio.montoAbono ? Number(servicio.montoAbono).toLocaleString('es-CO') : '',
    reparacion: Array.isArray(servicio.reparacion) ? servicio.reparacion : [servicio.reparacion].filter(Boolean),
    reparacionOtro: servicio.reparacionOtro || ''
  }
  actualizarModelos()

  indiceEditar.value = index
  editando.value = true
  mostrarModal.value = true
}

function confirmarEliminar(index) {
  if (index < 0 || index >= servicios.value.length || servicios.value[index].estadoEquipo === 'entregado') {
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
    servicios.value.splice(indiceEliminar.value, 1)
  }
  buscarServicios()
  cerrarConfirmacion()
}

// Inicialización de la búsqueda para mostrar todos los servicios al abrir la app
buscarServicios()
</script>

<style scoped>
.pagina {
  background: linear-gradient(180deg, #f8fbfc 0%, #eef4f5 100%);
  font-family: "Plus Jakarta Sans", "Avenir Next", "Trebuchet MS", sans-serif;
  min-height: 100vh;
  padding: 24px clamp(14px, 3.5vw, 48px) 48px;
}

.header {
  background: linear-gradient(115deg, #173f46 0%, #245c62 100%);
  box-shadow: 0 4px 18px rgba(18, 52, 58, 0.24);
}

.barra-superior {
  min-height: 72px;
  padding: 8px clamp(14px, 3.5vw, 48px);
}

.marca-header {
  display: flex;
  align-items: center;
  flex: 1;
  min-width: 0;
  gap: 14px;
}

.boton-nuevo {
  border-radius: 10px;
  padding: 0 18px;
  font-weight: 700;
  text-transform: none;
  letter-spacing: 0;
  margin-left: 14px;
}

.titulo {
  font-size: 20px;
  font-weight: bold;
  letter-spacing: 0.1px;
  color: white;
}

.subtitulo {
  font-size: 13px;
  opacity: 0.9;
  letter-spacing: 0.2px;
  color: #e0eaec;
}

/* Buscador y Controles de Navegación */
.buscador {
  border: 1px solid #e0eaec;
  border-radius: 14px;
  background: white;
  box-shadow: 0 6px 20px rgba(42, 76, 82, 0.06) !important;
}

.filtros-navegacion {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 6px;
  padding-top: 8px;
  border-top: 1px solid #f0f4f5;
}

.chips-container {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
}

.filtro-chip {
  font-weight: 600;
  font-size: 12px;
  padding: 4px 12px;
  position: relative;
}

.etiqueta-filtrar {
  font-size: 13px;
  color: #29626D;
}

/* Tarjetas de Estadísticas / Accesos directos */
.estadistica {
  border: 1px solid #e0eaec;
  border-radius: 14px;
  height: 100%;
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 6px 20px rgba(42, 76, 82, 0.05) !important;
  transition: transform 0.18s ease, box-shadow 0.18s ease, border-color 0.18s ease;
}

.estadistica:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 24px rgba(42, 76, 82, 0.1) !important;
  border-color: #245c62;
}

.tarjeta-activa {
  border: 2px solid #173f46 !important;
  background: #f4f9f9 !important;
}

.estadistica-contenido {
  display: flex;
  align-items: center;
  gap: 14px;
}

.texto-estadistica {
  color: #6b7280;
  font-size: 13px;
  font-weight: 500;
}

.numero {
  font-size: 26px;
  font-weight: bold;
  color: #164b58;
  line-height: 1.1;
}

.numero-dinero {
  font-size: 22px;
  color: #c62828;
}

.color-circle {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.color-circle.primary {
  background: #007bff;
}

.color-circle.warning {
  background: #f59e0b;
}

.color-circle.success {
  background: #10b981;
}

.color-circle.negative {
  background: #ef4444;
}

/* Tarjetas de Servicio Técnico */
.tarjeta {
  border: 1px solid #e0eaec;
  border-radius: 14px;
  overflow: hidden;
  align-self: flex-start;
  width: 100%;
  background: rgba(255, 255, 255, 0.98);
  box-shadow: 0 8px 22px rgba(42, 76, 82, 0.06) !important;
  transition: transform 0.18s ease, box-shadow 0.18s ease;
}

.tarjeta:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 28px rgba(42, 76, 82, 0.11) !important;
}

.pago-pendiente {
  border-left: 6px solid #c62828;
}

.pago-abono {
  border-left: 6px solid #f2a900;
}

.equipo-listo {
  border-top: 3px solid #10b981;
}

.equipo-entregado {
  opacity: 0.88;
  background: #fafbfc;
}

.nombre-cliente {
  font-size: 17px;
  font-weight: 700;
  color: #20383c;
}

.equipo {
  color: #5d6f72;
  font-size: 14px;
  line-height: 1.25;
  font-weight: 500;
}

.tarjeta-cabecera {
  padding: 14px 16px 10px;
}

.tarjeta-contenido {
  padding: 12px 16px 10px;
}

.badge-estado {
  border-radius: 8px;
  font-weight: 600;
  font-size: 12px;
}

.badge-pulse {
  animation: pulse-border 2s infinite;
}

@keyframes pulse-border {
  0% {
    box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.5);
  }
  70% {
    box-shadow: 0 0 0 6px rgba(16, 185, 129, 0);
  }
  100% {
    box-shadow: 0 0 0 0 rgba(16, 185, 129, 0);
  }
}

.datos-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px 14px;
}

.dato {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #4a5568;
  font-size: 14px;
  line-height: 1.3;
  min-width: 0;
}

.dato span {
  min-width: 0;
  overflow-wrap: anywhere;
}

.dato .q-icon {
  color: #29626D;
  font-size: 18px;
}

.titulo-pequeno {
  color: #64748b;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.estado-bloque {
  margin-top: 6px;
}

.estados-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
  padding: 8px 10px;
  background: #f8fafb;
  border-radius: 10px;
}

.deuda-texto {
  font-size: 12px;
  color: #c62828;
}

.observaciones {
  background: #f1f5f6;
  border-radius: 8px;
  padding: 8px 12px;
  color: #475569;
  border-left: 3px solid #29626D;
}

.tarjeta-acciones {
  padding: 6px 12px 10px;
  border-top: 1px solid #f0f4f5;
}

.mensaje-vacio {
  text-align: center;
  padding: 60px 20px;
  color: #64748b;
}

.mensaje-vacio h4 {
  margin: 12px 0 6px;
  color: #334155;
  font-size: 22px;
  font-weight: 700;
}

.mensaje-vacio p {
  margin-top: 4px;
  font-size: 15px;
}

/* Modales */
.modal {
  width: 620px;
  max-width: 95vw;
  border-radius: 14px;
  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;
}

.modal-header {
  padding: 14px 18px;
}

.modal-cuerpo {
  max-height: 75vh;
  overflow-y: auto;
}

.confirmacion {
  width: 440px;
  max-width: 95vw;
  border-radius: 14px;
  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;
}

.calificacion-modal {
  width: 420px;
  max-width: 95vw;
  border-radius: 14px;
  box-shadow: 0 18px 50px rgba(31, 67, 73, 0.18) !important;
}

@media (max-width: 600px) {
  .pagina {
    padding: 12px 10px 40px;
  }

  .titulo {
    font-size: 17px;
  }

  .subtitulo {
    display: none;
  }

  .boton-nuevo {
    font-size: 12px;
    padding: 0 12px;
  }

  .datos-grid {
    grid-template-columns: 1fr;
    gap: 6px;
  }

  .estados-grid {
    grid-template-columns: 1fr;
    gap: 8px;
  }

  .filtro-chip {
    font-size: 11px;
    padding: 2px 8px;
  }
}
</style>
