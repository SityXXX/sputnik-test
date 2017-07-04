<template>
    <div>
        <table class="table table-hover">
            <thead>
                <tr class="success">
                    <td>#</td>
                    <td>ПІП</td>
                    <td>Вік
                        <i v-if="sortConfig.type === 'age'
                        && sortConfig.direction === 'asc'
                        && customerList.length > 1"
                           class="fa fa-sort-asc"
                           @click="sortArr('desc', 'age')"></i>
                        <i v-if="sortConfig.type === 'age'
                        && sortConfig.direction === 'desc'
                        && customerList.length > 1"
                           class="fa fa-sort-desc"
                           @click="sortArr('asc', 'age')"></i>
                        <i v-if="sortConfig.type === 'location'
                        || customerList.length < 2"
                           class="fa fa-sort"
                           @click="sortArr('desc','age')"></i>
                    </td>
                    <td>Місто
                        <i v-if="sortConfig.type === 'location'
                        && sortConfig.direction === 'asc'
                        && customerList.length > 1"
                           class="fa fa-sort-asc"
                           @click="sortArr('desc', 'location')"></i>
                        <i v-if="sortConfig.type === 'location'
                        && sortConfig.direction === 'desc'
                        && customerList.length > 1"
                           class="fa fa-sort-desc"
                           @click="sortArr('asc', 'location')"></i>
                        <i v-if="sortConfig.type === 'age'
                        || customerList.length < 2"
                           class="fa fa-sort"
                           @click="sortArr('desc','location')"></i>
                    </td>
                    <td>Фото</td>
                    <td></td>
                </tr>
                </thead>
                <tbody class="table-striped">
                <tr v-for="(item, index) in getList">
                    <td>{{index + 1}}</td>
                    <td>{{item.suname}} {{item.name}} {{item.lastName}}</td>
                    <td>{{item.age}}</td>
                    <td>{{item.location}}</td>
                    <td><img :src="'http://lorempixel.com/50/50/people/'+(index+1)" alt="Фото"></td>
                    <td><i class="fa fa-trash" @click="deleteItem(item.id)"></i></td>
                </tr>
            </tbody>
        </table>
        <hr>
        <div class="row">
            <div class="col-xs-12 col-sm-8 col-sm-offset-2 col-md-6 col-md-offset-3">
                <form @submit.prevent>
                    <div class="form-group">
                        <label for="suname">Прізвище</label>
                        <input type="text" id="suname" class="form-control" v-model="userData.suname">
                    </div>
                    <div class="form-group">
                        <label for="name">Ім'я</label>
                        <input type="text" id="name" class="form-control" v-model="userData.name">
                    </div>
                    <div class="form-group">
                        <label for="lastname">По-батькові</label>
                        <input type="text" id="lastname" class="form-control" v-model="userData.lastName">
                    </div>
                    <div class="form-group">
                        <label for="age">Вік</label>
                        <input type="text" id="age" class="form-control" v-model="userData.age">
                    </div>
                    <div class="form-group">
                        <label for="location">Місто</label>
                        <input type="text" id="location" class="form-control" v-model="userData.location">
                    </div>
                    <button class="btn btn-primary" @click="sendData">Надіслати</button>
                </form>
            </div>
        </div>
    </div>
</template>
<script>
export default {

    // Data
    data() {
        return {
            customerList: '',
            userData: {
                name: '',
                suname: '',
                lastName: '',
                age: '',
                location: ''
            },
            sortConfig: {
                type: 'age',
                direction: 'desc'
            }
        }
    },

    // Get customer list on load
    mounted(){
        this.getData();
    },

    methods: {

        // Delete data from DB
        deleteItem(key) {
            this.$http.delete(key+'.json')
                .then(() => {
                   this.customerList = this.customerList.filter((el) => {
                       return el.id !== key;
                   });
                   this.sortArr(this.sortConfig.direction, this.sortConfig.type);
                });
        },

        // Post data to DB
        sendData() {
            this.$http.post('.json', this.userData)
                .then(respons => {
                    let id = respons.body.name;
                    let name = this.userData.name;
                    let suname = this.userData.suname;
                    let lastName = this.userData.lastName;
                    let age = this.userData.age;
                    let location = this.userData.location;
                    this.customerList.push({id: id, name: name, suname: suname, lastName: lastName, age: age, location: location});
                    this.sortArr(this.sortConfig.direction, this.sortConfig.type);
                    console.log(respons);
                }, error => {
                    console.log(error);
                });
        },

        // Get data from DB
        getData() {
            this.$http.get('.json')
                .then(respons => {
                    return respons.json();
                }, error => {
                    console.log(error);
                })
                .then(data => {
                    let usersArray = [];
                    for(let key in data) {
                        data[key]['id'] = key;
                        usersArray.push(data[key]);
                    }
                    this.customerList = usersArray;
                    this.sortArr(this.sortConfig.direction, this.sortConfig.type);
                });
        },

        // Sort data
        // [customerList: Array]
        // [direction: String]
        // [type: String]
        sortArr(direction = 'asc', type = 'age') {
            if(Array.isArray(this.customerList) && this.customerList.length > 1) {

                this.sortConfig.type = type;
                this.sortConfig.direction = direction;

                this.customerList.sort((a, b) => {
                    if (a[type] > b[type]) return 1;
                    if (a[type] < b[type]) return -1;
                    return 0;
                });

                if (direction === 'desc') {
                    this.customerList.reverse();
                }
            }

        }

    },

    // Computed value of customer list
    computed: {
        getList() {
            return this.customerList;
        }
    },

}
</script>

<style>
    [class*="fa"] {
        cursor: pointer;
    }
</style>