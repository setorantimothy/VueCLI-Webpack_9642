<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title primary-title>
                <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="search"
                single-line
                hide-details></v-text-field>
                <v-spacer></v-spacer>
                <v-select
                    :items="['All Priority','Penting', 'Biasa', 'Tidak Penting']"
                    v-model="filters"
                    label="Priority"
                    @change="filterPriority"
                    outlined
                    hide-details
                    class="mr-4"
                    dense></v-select>
                <v-btn color="success" dark @click="dialog = true">Tambah</v-btn>
            </v-card-title>
            
            <v-data-table 
            :headers="headers" 
            :items="filterPriority" 
            :search="search"
            pagination.sync="pagination">
                
                <template v-slot:[`item.actions`]="{ item }">
                    <v-icon small color="red" class="mr-2" @click="editItem(item)">
                    mdi-pencil
                    </v-icon>
                    <v-icon color="blue" small @click="dialogDelete = true">
                    mdi-delete
                    </v-icon>

                    <v-dialog v-model="dialogDelete" max-width="300px" transition="dialog-transition">
                        <v-card>
                            <v-card-title>Delete</v-card-title>
                            <v-card-text>Yakin delete item ?</v-card-text>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color="green darken-1"
                                    text
                                    @click="dialogDelete = false"
                                >
                                    TIDAK
                                </v-btn>
                                <v-btn
                                    color="red"
                                    text
                                    @click="deleteItem(item)"
                                >
                                    YA
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>

                </template>
                <template v-slot:[`item.selected`]="{ item }">
                    <v-checkbox v-model="selectedItem" :value="item.task"></v-checkbox>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog
            v-model="dialog"
            persistent
            max-width="600px"
            transition="dialog-transition"
        >
            <v-card>
                <v-card-title>
                    <span class="headline">From Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>
                        <v-select
                            :items="['Penting', 'Biasa', 'Tidak Penting']"
                            v-model="formTodo.priority"
                            label="All Priority"
                            required
                        ></v-select>
                        <v-textarea
                        v-model="formTodo.note" label="Note" required>
                        </v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogDeleteSelected" max-width="300px" transition="dialog-transition">
            <v-card>
                <v-card-title>Delete selected item</v-card-title>
                <v-card-text>Yakin delete item yang dipilih?</v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="green darken-1"
                        text
                        @click="dialogDeleteSelected = false"
                    >
                        TIDAK
                    </v-btn>
                    <v-btn
                        color="red"
                        text
                        @click="deleteSelectedItem"
                    >
                        YA
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        <v-card class="mt-5" v-if="selectedItem.length">
            <v-card-title primary-title>
                Delete Multiple
            </v-card-title>
            <v-card-text>
                <ul>
                    <li v-for="item in selectedItem">
                        {{ item }}
                    </li>
                </ul>
            </v-card-text>
            <v-card-actions>
                <v-btn color="error" @click="dialogDeleteSelected = true">HAPUS SEMUA</v-btn>
            </v-card-actions>
        </v-card>
    </v-main>
</template>
<script>
export default {
    name:"List",
    data(){
        return {
            search:null,
            dialog:false,
            dialogDelete:false,
            dialogDeleteSelected:false,
            filters:"All Priority",
            selectedItem: [],
            headers: [
                {
                    text:'Task',
                    align: 'start',
                    sortable: true,
                    value: 'task',
                },
                { text: "Priority", value:"priority"},
                { text: "Note", value:"note"},
                { text: "Actions", value:"actions"},
                { text: "", value:"selected"},
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "hufftts",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak Penting",
                    note: "bersama teman teman"
                },
            ],
            formTodo:  {
                task: null,
                priority: null,
                note: null,
            },
        };
    },
    methods: {
        save(){
            let index=  this.findIndexTodos(this.formTodo);
            if(index < 0){ // alias ga ketemu -1
                this.todos.push(this.formTodo);
            }else {
                this.todos[index] = this.formTodo;
            }
            this.resetForm();
            this.dialog = false;
        },
        cancel(){
            this.resetForm();
            this.dialog = false;
        },
        resetForm(){
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item){
            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
            this.dialog = true;
        },
        deleteItem(item){
            let index = this.findIndexTodos(item);
            this.todos.splice(index,1);
            this.dialogDelete = false;
        },
        findIndexTodos(item){
            return this.todos.findIndex(obj => obj.task === item.task);
        },
        deleteSelectedItem(){
            for (var i = 0 ; i < this.selectedItem.length; i++){
                let index = this.todos.findIndex(obj => obj.task === this.selectedItem[i])
                this.todos.splice(index,1);
            }
            this.dialogDeleteSelected = false;
            this.selectedItem = [];
        }
    },
    computed: {
        filterPriority(){
            let finds = this.filters;
            if(finds == "All Priority"){
                return this.todos;
            }else {
                var fil = this.todos.filter(function(x){
                    return x.priority == finds;
                })
                return fil;
            }
            
        },
    },
    
};
</script>