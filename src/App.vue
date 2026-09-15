<template>
  <q-layout view="lHh Lpr lFf">
    <!-- Header Original -->
    <q-header class="header" elevated>
      <q-toolbar class="barra-superior">
        <div class="marca-header cursor-pointer" @click="resetearFiltros">
          <q-avatar
            color="white"
            text-color="primary"
            size="44px"
          >
            <q-icon name="build" size="24px" />
          </q-avatar>
          <q-toolbar-title>
            <div class="titulo">
              Servicio Técnico
            </div>
            <div class="subtitulo">
              Gestión de celulares y tablets - Don Efraín
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

        <!-- Tarjetas de Estadísticas / Accesos Rápidos -->
        <div class="row q-col-gutter-md q-mb-md">
          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer"
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

          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer"
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

          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer"
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

          <div class="col-12 col-sm-6 col-md-3">
            <q-card
              class="estadistica cursor-pointer"
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

        <!-- Buscador y Filtros de Navegación -->
        <q-card class="buscador q-mb-lg">
          <q-card-section class="q-pa-md">
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
                  placeholder="Ej: Danna, Samsung A15, iPhone, pantalla..."
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
                >
                  <template v-slot:prepend>
                    <q-icon name="sort" color="primary" />
                  </template>
                </q-select>
              </div>
            </div>

            <!-- Chips de Filtros Rápidos (Sin numeritos) -->
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
                  @click="resetearFiltros"
                />
              </div>
            </div>
          </q-card-section>
        </q-card>

        <!-- Mensaje cuando no hay registros -->
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
            class="q-mt-md"
            @click="abrirNuevoServicio"
          />
        </div>

        <!-- Listado de Tarjetas -->
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
                'pago-abono': resultado.servicio.estadoPago === 'abono',
                'equipo-entregado': resultado.servicio.estadoEquipo === 'entregado',
                'equipo-listo': resultado.servicio.estadoEquipo === 'listo para entregar'
              }"
            >
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
                      class="q-pa-xs badge-estado"
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

              <q-card-section class="tarjeta-contenido">
                <div class="datos-grid">
                  <div class="dato">
                    <q-icon name="build" />
                    <span>
                      {{ textoReparaciones(resultado.servicio) }}
                    </span>
                  </div>

                  <div class="dato">
                    <q-icon name="person" />
                    <span>
                      Técnico: <strong>{{ resultado.servicio.tecnico }}</strong>
                    </span>
                  </div>

                  <div class="dato">
                    <q-icon name="schedule" />
                    <span>
                      {{ formatoFecha(resultado.servicio.fecha) }}
                    </span>
                  </div>

                  <div class="dato">
                    <q-icon name="attach_money" />
                    <span>
                      Total: <strong>{{ formatoPesos(resultado.servicio.precio) }}</strong>
                    </span>
                  </div>

                  <div class="dato">
                    <q-icon name="payments" />
                    <span>
                      Pago: {{ resultado.servicio.metodoPago }}
                    </span>
                  </div>

                  <div class="dato" v-if="resultado.servicio.telefono">
                    <q-icon name="phone" />
                    <span>
                      Tel: {{ resultado.servicio.telefono }}
                    </span>
                  </div>
                </div>

                <div class="estados-grid q-mt-md">
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

                      <!-- Avance rápido de estado -->
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
                      />
                    </div>
                  </div>

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
                        ✓ Pagado
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

              <q-card-actions align="between" class="tarjeta-acciones">
                <div>
                  <q-btn
                    v-if="resultado.servicio.telefono && resultado.servicio.estadoEquipo === 'listo para entregar'"
                    flat
                    round
                    dense
                    color="positive"
                    icon="chat"
                    title="Avisar por WhatsApp"
                    @click="abrirWhatsApp(resultado.servicio)"
                  />
                </div>

                <div class="row items-center q-gutter-xs">
                  <q-btn
                    v-if="resultado.servicio.estadoEquipo === 'entregado' && resultado.servicio.estadoPago === 'pagado' && !resultado.servicio.calificacion"
                    flat
                    icon="star"
                    label="Calificar"
                    color="amber-9"
                    size="sm"
                    @click="abrirCalificacion(resultado.servicio)"
                  />

                  <q-btn
                    v-if="resultado.servicio.estadoEquipo !== 'entregado'"
                    flat
                    icon="edit"
                    label="Editar"
                    color="primary"
                    size="sm"
                    @click="editarServicio(resultado.indice)"
                  />

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

        <!-- Botón flotante para celular -->
        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn
            fab
            icon="add"
            color="primary"
            text-color="white"
            @click="abrirNuevoServicio"
          />
        </q-page-sticky>

        <!-- Modal Formulario Nuevo / Editar -->
        <q-dialog v-model="mostrarModal" persistent>
          <q-card class="modal">
            <q-card-section class="row items-center modal-header">
              <div class="text-h6 text-weight-bold">
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
              <q-card-section class="modal-cuerpo q-pa-md">
                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  dense
                  :rules="[reglaNombre]"
                  class="q-mb-sm"
                />

                <q-input
                  v-model="formulario.telefono"
                  label="Teléfono / Celular (opcional)"
                  placeholder="Ej: 3101234567"
                  outlined
                  dense
                  class="q-mb-sm"
                />

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
                />

                <q-input
                  v-if="formulario.reparacion.includes('Otros')"
                  v-model="formulario.reparacionOtro"
                  label="Especifica el otro tipo de servicio *"
                  placeholder="Ej: Reparación de cámara"
                  outlined
                  dense
                  :rules="[reglaOtro]"
                  class="q-mb-sm"
                />

                <q-select
                  v-model="formulario.tecnico"
                  label="Técnico que atendió *"
                  outlined
                  dense
                  :options="tecnicos"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                />

                <q-input
                  :model-value="formatoFecha(formulario.fecha)"
                  label="Fecha y hora de recepción *"
                  outlined
                  dense
                  readonly
                  class="q-mb-sm"
                />

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
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select
                      v-model="formulario.metodoPago"
                      label="Método de pago *"
                      outlined
                      dense
                      :options="metodosPago"
                      :rules="[reglaObligatoria]"
                    />
                  </div>
                </div>

                <q-select
                  v-model="formulario.estadoPago"
                  label="Estado del pago *"
                  outlined
                  dense
                  :options="estadosPago"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                />

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
                />

                <q-select
                  v-model="formulario.estadoEquipo"
                  label="Estado del equipo *"
                  outlined
                  dense
                  :options="estadosEquipo"
                  :disable="!editando"
                  :rules="[reglaObligatoria]"
                  class="q-mb-sm"
                />

                <div
                  v-if="mostrarErrorEntrega"
                  class="q-pa-sm bg-red-1 text-negative rounded-borders q-mb-sm text-caption"
                >
                  Para entregar el equipo al cliente, el estado del pago debe ser <strong>pagado</strong>.
                </div>

                <q-input
                  v-model="formulario.observaciones"
                  label="Observaciones (opcional)"
                  outlined
                  dense
                  type="textarea"
                  rows="2"
                  placeholder="Ej: Pantalla partida en la esquina superior..."
                />
              </q-card-section>

              <q-separator />

              <q-card-actions align="right" class="q-pa-md">
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

        <!-- Modal Calificación -->
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

            <q-separator />

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

        <!-- Modal Confirmación Eliminación -->
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

            <q-separator />

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
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Datos iniciales de ejemplo
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
    observaciones: 'Pantalla con líneas verdes. Cliente pide respaldar fotos.'
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
    observaciones: 'Batería inflada, no retiene carga.'
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
    observaciones: 'Pin sulfatado reemplazado con éxito.'
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
    observaciones: 'Equipo entregado a satisfacción.'
  }
]

// Persistencia obligatoria con useLocalStorage
const servicios = useLocalStorage('servicios-tecnicos', datosEjemplo)

// Estados con ref() únicamente
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

// Estados de filtrado y orden
const filtroEstado = ref('todos')
const filtroPago = ref('todos')
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

// Reglas de validación Quasar
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
    return 'El abono debe ser menor al precio total'
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
  return `${dia}/${mes}/${anio}${hora ? ` ${hora.slice(0, 5)}` : ''}`
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
  return precio
}

function calcularTotalPendienteCobro() {
  let total = 0
  for (let i = 0; i < servicios.value.length; i++) {
    total += calcularDeuda(servicios.value[i])
  }
  return total
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

function iconoPorMarca(marca) {
  const m = String(marca || '').toLowerCase()
  if (m === 'apple') return 'phone_iphone'
  if (m === 'tablet' || m === 'ipad') return 'tablet_android'
  return 'phone_android'
}

// Búsqueda y filtrado con ref() y funciones normales
const serviciosFiltrados = ref([])

function buscarServicios() {
  let resultados = []
  const texto = String(busqueda.value || '').toLowerCase().trim()

  for (let i = 0; i < servicios.value.length; i++) {
    const serv = servicios.value[i]

    if (filtroEstado.value !== 'todos' && serv.estadoEquipo !== filtroEstado.value) {
      continue
    }

    if (filtroPago.value === 'con-saldo') {
      if (serv.estadoPago !== 'pendiente' && serv.estadoPago !== 'abono') {
        continue
      }
    }

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

// Avance rápido de estado
function avanzarEstadoRapido(indice) {
  if (indice < 0 || indice >= servicios.value.length) return
  const serv = servicios.value[indice]

  if (serv.estadoEquipo === 'recibido') {
    serv.estadoEquipo = 'en reparación'
  } else if (serv.estadoEquipo === 'en reparación') {
    serv.estadoEquipo = 'listo para entregar'
  } else if (serv.estadoEquipo === 'listo para entregar') {
    if (serv.estadoPago !== 'pagado') {
      editarServicio(indice)
      mostrarErrorEntrega.value = true
      return
    }
    serv.estadoEquipo = 'entregado'
    abrirCalificacion(serv)
  }
  buscarServicios()
}

function abrirWhatsApp(servicio) {
  if (!servicio.telefono) return
  const telefonoLimpio = String(servicio.telefono).replace(/[^0-9]/g, '')
  const mensaje = encodeURIComponent(
    `Hola ${servicio.cliente}, te escribimos del taller de Don Efraín. Tu equipo (${servicio.marca} ${servicio.modelo || ''}) ya está listo para entregar. Saldo pendiente: ${formatoPesos(calcularDeuda(servicio))}. ¡Te esperamos!`
  )
  window.open(`https://wa.me/57${telefonoLimpio}?text=${mensaje}`, '_blank')
}

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
    servicios.value.unshift(servicio)
  }

  buscarServicios()

  mostrarModal.value = false
  limpiarFormulario()
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

// Inicialización automática
buscarServicios()
</script>

<style scoped>
.pagina {
  background: linear-gradient(180deg, #f8fbfc 0%, #eef4f5 100%);
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
}

.etiqueta-filtrar {
  font-size: 13px;
  color: #29626D;
}

.estadistica {
  border: 1px solid #e0eaec;
  border-radius: 14px;
  height: 100%;
  background: rgba(255, 255, 255, 0.94);
  box-shadow: 0 8px 22px rgba(42, 76, 82, 0.06) !important;
  transition: transform 0.18s ease, border-color 0.18s ease;
}

.estadistica:hover {
  transform: translateY(-2px);
  border-color: #245c62;
}

.tarjeta-activa {
  border: 2px solid #173f46 !important;
  background: #f4f9f9 !important;
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

.numero-dinero {
  font-size: 24px;
  color: #c62828;
}

.color-circle {
  width: 50px;
  height: 50px;
  border-radius: 14px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.color-circle.primary {
  background: #007bff;
  border: 1px solid #0066d6;
}

.color-circle.warning {
  background: #fed843;
  border: 1px solid #e5bd26;
}

.color-circle.success {
  background: #10b981;
  border: 1px solid #059669;
}

.color-circle.negative {
  background: #ec1000;
  border: 1px solid #c90e00;
}

.tarjeta {
  border: 1px solid #e0eaec;
  border-radius: 14px;
  overflow: hidden;
  align-self: flex-start;
  width: 100%;
  background: rgba(255, 255, 255, 0.96);
  box-shadow: 0 9px 24px rgba(42, 76, 82, 0.07) !important;
  transition: transform 0.18s ease;
}

.tarjeta:hover {
  transform: translateY(-2px);
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
  opacity: 0.9;
  background: #fafbfc;
}

.nombre-cliente {
  font-size: 18px;
  font-weight: bold;
  color: #20383c;
}

.equipo {
  color: #6b7280;
  font-size: 15px;
  line-height: 1.25;
}

.tarjeta-cabecera {
  padding: 14px 16px 12px;
}

.tarjeta-contenido {
  padding: 14px 16px 12px;
}

.badge-estado {
  border-radius: 8px;
  font-weight: 600;
  font-size: 12px;
}

.datos-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 8px 18px;
}

.dato {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 0;
  color: #555;
  font-size: 15px;
  line-height: 1.3;
  min-width: 0;
}

.dato span {
  min-width: 0;
  overflow-wrap: anywhere;
}

.dato .q-icon {
  color: #29626D;
}

.titulo-pequeno {
  color: #6b7280;
  font-size: 14px;
  font-weight: 600;
  line-height: 1.25;
}

.estado-bloque {
  margin-top: 10px;
}

.estados-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.deuda-texto {
  font-size: 12px;
  color: #c62828;
}

.observaciones {
  background: #f1f3f4;
  border-radius: 10px;
  padding: 10px 12px;
  color: #555;
}

.tarjeta-acciones {
  padding: 6px 12px 10px;
  border-top: 1px solid #f0f4f5;
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
  .datos-grid {
    grid-template-columns: 1fr;
  }
  .estados-grid {
    grid-template-columns: 1fr;
  }
}
</style>
