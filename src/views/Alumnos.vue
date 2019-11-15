<template>
  <div class="container">
    <h1>Alumnos</h1>
    <b-alert
      :show="dismissCountDown"
      dismissible
      :variant="mensaje.color"
      @dismissed="dismissCountDown=0"
      @dismiss-count-down="countDownChanged"
    >{{mensaje.texto}}</b-alert>

    <form @submit.prevent="actualizarAlumno(alumnoEditar)" v-if="editar">
      <h3 class="text-center">Editar y Actualizar Alumno</h3>
      <input
        type="text"
        placeholder="Ingrese el Apellido"
        class="form-control my-2"
        v-model="alumnoEditar.apellido"
      />
      <input
        type="text"
        placeholder="Ingrese el Nombre"
        class="form-control my-2"
        v-model="alumnoEditar.nombre"
      />
      <input
        type="text"
        placeholder="Ingrese el DNI"
        class="form-control my-2"
        v-model="alumnoEditar.dni"
      />
      <input
        type="text"
        placeholder="Ingrese una descripcion"
        class="form-control my-2"
        v-model="alumnoEditar.descripcion"
      />
      <b-button class="btn-sm my-2 mx-2 btn-warning" type="submit">Actualizar</b-button>
      <b-button class="btn-sm my-2" type="submit" @click="editar = false">Cancelar</b-button>
    </form>

    <form @submit.prevent="agregarAlumno()" v-if="!editar">
      <h3 class="text-center">Agregar nuevo Alumno</h3>
      <input
        type="text"
        placeholder="Ingrese el Apellido"
        class="form-control my-2"
        v-model="alumno.apellido"
      />
      <input
        type="text"
        placeholder="Ingrese el Nombre"
        class="form-control my-2"
        v-model="alumno.nombre"
      />
      <input
        type="text"
        placeholder="Ingrese el DNI"
        class="form-control my-2"
        v-model="alumno.dni"
      />
      <input
        type="text"
        placeholder="Ingrese una descripcion"
        class="form-control my-2"
        v-model="alumno.descripcion"
      />
      <b-button class="btn-sm btn-block btn-success" type="submit">Agregar</b-button>
    </form>

    <table class="table">
      <thead>
        <tr>
          <th scope="col">#</th>
          <th scope="col">Apellido</th>
          <th scope="col">Nombre</th>
          <th scope="col">DNI</th>
          <th scope="col">Descripción</th>
          <th scope="col">Fecha</th>
          <th scope="col">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in alumnos" :key="index">
          <th scope="row">{{item._id}}</th>
          <td>{{item.apellido}}</td>
          <td>{{item.nombre}}</td>
          <td>{{item.dni}}</td>
          <td>{{item.descripcion}}</td>
          <td>{{item.date}}</td>
          <td>
            <b-button class="btn-warning btn-sm mx-2" @click="activarEdicion(item._id)">Actualizar</b-button>
            <b-button class="btn-danger btn-sm" @click="eliminarAlumno(item._id)">Eliminar</b-button>
            <!--<b-button class="btn-danger btn-sm mx-2" @click="alerta()">Accion</b-button>-->
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      alumnos: [],
      mensaje: { color: "success", texto: "" },
      dismissSecs: 5,
      dismissCountDown: 0,
      alumno: { apellido: "", nombre: "", dni: "", descripcion: "" },
      editar: false,
      alumnoEditar: {}
    };
  },
  created() {
    this.listarAlumnos();
  },
  methods: {
    alerta() {
      this.mensaje.color = "danger";
      this.mensaje.texto = "Probando alerta de bootstrap";
      this.showAlert();
    },
    listarAlumnos() {
      this.axios
        .get("/alumno")
        .then(response => {
          console.log(response.data);
          this.alumnos = response.data;
        })
        .catch(e => {
          //console.log('error' + e.response);
          console.log(e.response);
        });
    },
    agregarAlumno() {
      //console.log(this.alumno);
      this.axios
        .post("/nuevo-alumno", this.alumno)
        .then(res => {
          this.alumnos.push(res.data);
          this.alumno.apellido = "";
          this.alumno.nombre = "";
          this.alumno.dni = "";
          this.alumno.descripcion = "";
          this.mensaje.color = "success";
          this.mensaje.texto = "Alumno agregado correctamente";
          this.showAlert();
        })
        .catch(e => {
          console.log(e.response);
          if (e.response.data.error.errors.apellido.message) {
            this.mensaje.texto = e.response.data.error.errors.apellido.message;
          } else {
            this.mensaje.texto = "Alumno NO agregado por error de sistema";
          }
          this.mensaje.color = "danger";
          this.showAlert();
        });
    },
    eliminarAlumno(id) {
      console.log(id);
      //this.axios.delete('/nota/${id}')---OJO CON LAS COMILLAS---
      this.axios
        .delete(`/alumno/${id}`)
        .then(res => {
          //Para no actualizar el sitio web, se utiliza el "findIndex":
          const index = this.alumnos.findIndex(
            item => item._id === res.data._id
          );
          this.alumnos.splice(index, 1);
          this.mensaje.color = "success";
          this.mensaje.texto = "Alumno eliminado correctamente";
          this.showAlert();
        })
        .catch(e => {
          console.log(e.response);
        });
    },
    activarEdicion(id){
      this.editar = true;      
      console.log(id);
      this.axios.get(`/alumno/${id}`)
        .then(res => {
          this.alumnoEditar = res.data;
        })
        .catch(e => {
          console.log(e.response);
        })
    },
    actualizarAlumno(item){
      this.axios.put(`/alumno/${item._id}`, item)
        .then(res => {
          const index = this.alumnos.findIndex(n => n._id === res.data._id);
          this.alumnos[index].apellido = res.data.apellido;
          this.alumnos[index].nombre = res.data.nombre;
          this.alumnos[index].dni = res.data.dni;
          this.alumnos[index].descripcion = res.data.descripcion;
          this.mensaje.color = "success";
          this.mensaje.texto = "Alumno actualizado correctamente";
          this.showAlert();
          this.editar = false;
        })
        .catch(e => {
          console.log(e.response);
        })
    },
    countDownChanged(dismissCountDown) {
      this.dismissCountDown = dismissCountDown;
    },
    showAlert() {
      this.dismissCountDown = this.dismissSecs;
    }
  }
};
</script>