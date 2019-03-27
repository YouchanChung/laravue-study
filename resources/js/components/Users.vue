<template>
    <div class="container">

        <div class="row ">
            <div class="col-md-12 mt-3">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal()">Add New <i class="fa fa-user-plus fa-fw"></i></button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Registered at</th>
                                <th>Modify</th>
                            </tr>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td><span class="tag tag-success">{{user.type | upText}}</span></td>
                                <td>{{user.created_at | myDate}}</td>
                                <td>
                                    <a href="#" @click="editModal(user)"><i class="fa fa-edit blue"></i></a>
                                    /
                                    <a href="#" @click="deleteUser(user.id)"><i class="fa fa-trash red"></i></a>
                                </td>
                            </tr>

                            </tbody></table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" id="addNewLabel">{{ editmode ? "Update User's info" : "Add New"}}</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name"
                                   placeholder="Name"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>
                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email"
                                   placeholder="Email"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>
                        <div class="form-group">
                            <textarea v-model="form.bio"  name="bio" id="bio"
                                   placeholder="Short bio for user (Optional)"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                            </textarea>
                            <has-error :form="form" field="bui"></has-error>
                        </div>
                        <div class="form-group">
                            <select v-model="form.type" name="type" id="type"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password" id="password"
                                   placeholder="Password"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                            <has-error :form="form" field="password"></has-error>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                        <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
                    </div>
                    </form>
                </div>
            </div>
        </div>

    </div>


</template>

<script>
    export default {
        data() {
            return {
                editmode: false,
                title: 'Add  a  New',
                users: {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            newModal() {
                this.editmode = false;
                this.title = 'Add New';
                this.form.reset();
                $('#addNew').modal('show');
            },
            editModal(user) {
                this.editmode = true;
                this.title = 'Edit User';
                this.form.clear();
                $('#addNew').modal('show');
                this.form.fill(user);
            },
            deleteUser(id) {
                swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    // Send request to the server
                    if (result.value) {
                        this.form.delete('api/user/'+id).then(() => {
                            swal.fire(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            )
                            this.$emit('AfterCreate');

                        }).catch(() => {
                            swal.fire('Failed!', 'There was something wronge.", "warning"');
                        });
                    }

                })
            },
            loadUsers() {
                axios.get('api/user').then(({ data }) => (this.users = data.data));
            },
            createUser() {

                this.$Progress.start();

                this.form.post('api/user').then(() => {

                    $('#addNew').modal('hide');
                    this.$emit('AfterCreate');
                    toast.fire({
                        type: 'success',
                        title: 'User created in successfully'
                    });
                    this.$Progress.finish();

                }).catch(() => {
                    this.$Progress.fail();
                });
            },
            updateUser() {
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id).then(() => {
                    $('#addNew').modal('hide');
                    this.$emit('AfterCreate');
                    swal.fire(
                        'Updated!',
                        'Infomation has been updated.',
                        'success'
                    )
                    this.$Progress.finish();

                }).catch(() => {
                    this.$Progress.fail();
                });
            }

        },
        created() {
            this.loadUsers();
            this.$on('AfterCreate', () => {
                console.log('on after create listener : this');
                this.loadUsers();
            });

            // setInterval(() => this.loadUsers(), 3000);
        },
    }
</script>
