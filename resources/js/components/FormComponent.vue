<template>
    <div>
        <form @submit.prevent="editarTarea(tarea)" v-if="editarActivo">
            <h3>Editar Tarea</h3>
            <input type="text" placeholder="Nombre" class="form-control mb-2"
                v-model="tarea.nombre">
            <input type="text" placeholder="Descripción" class="form-control mb-2"
                v-model="tarea.descripcion">

            <button class="btn btn-success" type="submit" >Guardar</button>
            <button class="btn btn-danger" type="submit" 
            @click="cancelarEdicion()">Cancelar</button>
        </form>
        <form @submit.prevent="agregar" v-else>
            <h3>Agregar Tarea</h3>
            <input type="text" placeholder="Nombre" class="form-control mb-2"
                v-model="tarea.nombre">
            <input type="text" placeholder="Descripción" class="form-control mb-2"
                v-model="tarea.descripcion">

            <button class="btn btn-primary" type="submit" >Agregar</button>
        </form>
        <hr class="mt-3" >
        <h3>Listado de tareas</h3>
        <ul class="list-group my-2">
            <li class="list-group-item"
            v-for="(item, index) in tareas" :key="index">
                <span class="badge badge-primary float-right" >
                    {{ item.updated_at }}
                </span>
                <p><b>{{ item.nombre }}</b></p>
                <p> {{ item.descripcion }} </p>
                <button class="btn btn-danger btn-sm" 
                @click="eliminar(item, index)">
                    Eliminar
                </button>
                <button class="btn btn-warning btn-sm" 
                @click="editarFormulario(item, index)">
                    Editar
                </button>
            </li>
        </ul>

    </div>
    
</template>

<script>
    export default {
        data(){
            return{
                tareas: [],
                tarea: {nombre: '', descripcion: ''},
                editarActivo: false
            }
            
        },
        created(){
            axios.get('/notas')
            .then(res => {
                this.tareas = res.data;
            })
        },
        methods:{
            agregar(){
                if(this.tarea.nombre.trim() === '' || this.tarea.descripcion.trim() === ''){
                    alert('Debes completat todos los campos antes de guardar');
                    return;
                }
                console.log('nombre', this.tarea.nombre);
                console.log('descripcion', this.tarea.descripcion);
                const params = {
                    nombre: this.tarea.nombre,
                    descripcion: this.tarea.descripcion
                }

                this.tarea.nombre = '';
                this.tarea.descripcion = '';
                axios.post('/notas', params)
                    .then(res => {
                        this.tareas.push(res.data)
                    })
            },
            editarFormulario(item){

                this.editarActivo = true;
                this.tarea.nombre = item.nombre;
                this.tarea.descripcion = item.descripcion;
                this.tarea.id = item.id;
            },
            editarTarea(item){

                const params = {
                    nombre: item.nombre,
                    descripcion: item.descripcion
                }

                axios.put(`/notas/${item.id}`, params)
                .then(res=>{
                    this.editarActivo = false;
                    const index = this.tareas.findIndex(
                        notaBuscar => notaBuscar.id === res.data.id)
                        console.log('index', index);
                    this.tareas[index] = res.data;

                    this.tarea = {nombre: '', descripcion: ''};

                    axios.get('/notas')
                    .then(res => {
                        this.tareas = res.data;
                    })

                })
                
            },
            cancelarEdicion(){
                this.editarActivo = false;
                this.tarea = {nombre: '', descripcion: ''};
            }, 
            eliminar(item, index){
                axios.delete(`/notas/${item.id}`)
                .then(() =>{
                    this.tareas.splice(index, 1)
                })
            }
        }
    }
</script>