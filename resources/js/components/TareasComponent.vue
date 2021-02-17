<template>
<div>

    <form @submit.prevent="editarNota(nota)" v-if="editarActivo">
        <h3>Editar Nota</h3>
        <input type="text" placeholder="Nombre" v-model="nota.nombre" class="form-control mb-2">
        <input type="text" placeholder="Descripción" v-model="nota.descripcion" class="form-control mb-2">
        <button class="btn btn-primary" type="submit">Guardar</button>
        <button class="btn btn-primary" @click="cancelarEdicion()" type="submit">Cancelar</button>
    </form>

    <form @submit.prevent="agregar" v-else>
        <h3>Agregar Nota</h3>
        <input type="text" placeholder="Nombre" v-model="nota.nombre" class="form-control mb-2">
        <input type="text" placeholder="Descripción" v-model="nota.descripcion" class="form-control mb-2">
        <button class="btn btn-primary" type="submit">Agregar</button>
    </form>

    <ul class="list-group my-2">
        <li class="list-group-item" v-for="(item,index) in notas" :key="index">

            <span class="badge badge-primary float-right">
                {{item.updated_at}}
            </span>
            <p>{{item.nombre}}</p>
            <p>{{item.descripcion}}</p>
            <button class="btn btn-danger btn-sm" @click="eliminarNota(item,index)">Eliminar</button>
            <button class="btn btn-warning btn-sm" @click="editarFormulario(item)">Editar</button>
        </li>
    </ul>
</div>
</template>

<script>
export default {
    data() {
        return {
            notas: [],
            editarActivo: false,
            nota: {
                nombre: '',
                descripcion: '',
                id: -1
            },
            notaDefault: {
                nombre: '',
                descripcion: '',
                id: -1
            }
        }
    },
    created() {
        this.initialize();
    },
    methods: {
        initialize() {
            axios.get('/notas').then(res => {
                this.notas = res.data;
            });
        },
        //store
        agregar() {
            if (this.nota.nombre.trim() === '' ||
                this.nota.descripcion.trim() === '') {
                alert('Debes completar los datos antes de guardar');
                return;
            }
            const params = {
                nombre: this.nota.nombre,
                descripcion: this.nota.descripcion
            };
            this.nota = Object.assign({}, this.notaDefault)

            axios.post('/notas', params).then(
                res => {
                    this.notas.push(res.data);
                }
            );
        },

        eliminarNota(nota, index) {
            axios.delete(`/notas/${nota.id}`).then(() => {
                this.notas.splice(index, 1);
            });
        },

        editarFormulario(item) {
            this.editarActivo = true;
            this.nota.nombre = item.nombre;
            this.nota.id = item.id;
            this.nota.descripcion = item.descripcion;
        },

        editarNota(item) {
            const params = {
                nombre: item.nombre,
                descripcion: item.descripcion
            };
            axios.put(`/notas/${item.id}`, params).then(
                res => {
                    const index = this.notas.findIndex(
                        notaBuscar => notaBuscar.id === res.data.id);
                    this.nota = Object.assign({}, this.notaDefault);
                    this.notas[index].nombre = res.data;
                    this.initialize();
                    this.editarActivo = false;
                }
            )
        },
        cancelarEdicion() {
            this.editarActivo = false;
            this.nota = Object.assign({}, this.notaDefault);
        },
    }
}
</script>
