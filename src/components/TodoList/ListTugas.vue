<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                <v-text-field
                    v-model="search"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details>
                </v-text-field>
                <v-spacer></v-spacer>
                <v-select
                    v-model="sort"
                    @change="sortPriority"
                    label="Urutkan"
                    outlined
                    :items="['Penting','Tidak Penting']"
                    hide-details>
                </v-select>
                <v-spacer></v-spacer>
                <v-btn color="success" dark @click="dialog = true">Tambah</v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="todos" :search="search" item-key="note" :expanded.sync="expanded" show-expand>
                <template v-slot:expanded-item="{ headers, item }">
                    <td :colspan="headers.length">
                        {{ item.note }}
                    </td>
                </template>
                <template v-slot:[`item.priority`]="{ item }">
                    <v-chip
                        v-if="item.priority == 'Biasa'" class="ma-2" label outlined color="blue">Biasa</v-chip>
                    <v-chip
                        v-if="item.priority == 'Penting'" class="ma-2" label outlined color="red">Penting</v-chip>
                    <v-chip
                        v-if="item.priority == 'Tidak penting'" class="ma-2" label outlined color="green">Tidak Penting</v-chip>
                </template>
                <!-- v-slot untuk mengubh nilai dalam table, karena defaultnya hanya text, dan faktanya membutuhkan button (action) -->
                <template v-slot:[`item.actions`]="{ item }"> 
                    <v-icon small color="blue" class="mr-2" @click="editItem(item)"> mdi-pencil</v-icon>
                    <v-icon small color="red" @click="deleteItem(item)"> mdi-delete</v-icon>
                </template>

                <template v-slot:[`item.check`]="{ item }">
                    <input type="checkbox" v-model="checkedItem" :value="item.task">
                </template>
            </v-data-table>
        </v-card>

        <div v-if="checkedItem.length > 0">
            <v-card class="my-5">
                <v-card-text>
                    <p v-show="checkedItem" style="text-align:left; margin-left:20px">Delete Multiple:</p>
                    <ul v-for="(item, index) in checkedItem" :key="index" style="text-align:left; margin-left:20px">
                        <li>{{ item }}</li>
                    </ul>
                    <v-btn depressed color="error" style="margin-right:400px; margin-top: 15px;" @click="deleteChecked">Hapus Semua</v-btn>
                </v-card-text>
            </v-card>
        </div>

        <v-dialog v-model="dialogDelete" persistent max-width="300px">
                    <v-card>
                <v-card-title>Yakin ingin hapus ?</v-card-title>
                        <v-btn color="blue darken-1" text @click="cancelDelete()">Tidak</v-btn>
                        <v-btn color="blue darken-1" text @click="deleteItemDialog(indexItemDelete)">Ya</v-btn>
                    </v-card>
        </v-dialog>

        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                            v-model="formTodo.task"
                            label="Task"
                            required
                        ></v-text-field>

                        <v-select
                            v-model="formTodo.priority"
                            :items="['Penting','Biasa','Tidak penting']"
                            label="Priority"
                            required
                        ></v-select>

                        <v-textarea
                        v-model="formTodo.note"
                        label="Note"
                        required></v-textarea>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>
<script>
export default{
    name: "List",
    data(){
        return{
            search: null,
            dialog: false,
            dialogDelete: false,
            sort: null,
            isEdit: 0,
            beforeEditing: null,
            expanded:[],
            checkedItem:[],
            indexItemDelete:"",
            singleExpand: false,
            headers: [
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value: "priority" },
                { text: "Actions", value: "actions" },
                { text: "", value: "check" },
            ],
            todos: [
                {
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama tman2",
                },
                {
                    task: "masak",
                    priority: "Biasa",
                    note: "masak air 500ml",
                },
            ],
            formTodo: {
                task: null,
                priority: null,
                note: null,
            },
        };
    },
    methods: {
        save(){
            if(this.isEdit ==0){
                this.todos.push(this.formTodo);
                this.resetForm();
                this.dialog = false;
                this.sortPriority;
            }else{
                this.resetForm();
                this.dialog=false;
                this.isEdit = 0;
                this.sortPriority;
            }
            
        },
        cancel(){
            if(this.isEdit==1){
                this.todos[this.todos.findIndex(obj => obj.task === this.formTodo.task)].task = this.beforeEditing.task;
                this.todos[this.todos.findIndex(obj => obj.task === this.formTodo.task)].priority = this.beforeEditing.priority;
                this.todos[this.todos.findIndex(obj => obj.task === this.formTodo.task)].note = this.beforeEditing.note;
                this.beforeEditing = null;
                this.isEdit = 0;
            }
            this.resetForm();
            this.dialog=false;
        },
        resetForm(){
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        editItem(item){
            var itemEdit={
                task: item.task,
                priority: item.priority,
                note: item.note,
            };

            this.beforeEditing = itemEdit;
            this.formTodo = item;
            this.isEdit = 1;
            this.dialog = true;
        },
        deleteItem(item){
            var index = this.todos.findIndex(obj => obj.task === item.task);
            this.dialogDelete = true;
            this.indexItemDelete = index;
        },
        cancelDelete(){
            this.dialogDelete=false;
        },
        deleteItemDialog(index){
            this.todos.splice(index,1);
            this.indexItemDelete=null;
            this.dialogDelete = false;
        },
        sortPriority(){
            if(this.sort == "Penting"){
                this.todos.sort((todos)=>todos.priority == "Tidak penting" ? 1 : -1);
            }
            else{
                this.todos.sort((todos)=>todos.priority == "Penting" ? 1 : -1);
            }
        },
        deleteChecked(){
            var i;
            for(i=0;i<this.checkedItem.length;i++){
                var index = this.todos.findIndex(obj => obj.task === this.checkedItem[i]);
                this.todos.splice(index,1);
            }
            this.checkedItem =[];
        },
    }
};
</script>