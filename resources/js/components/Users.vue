<template>
    <div class="container">
      <div class="row">
        <div class="col-md-12 mt-5">
          <div class="box">
            <div class="box-header">
              <h3 class="box-title">Users Table</h3>

              <div class="box-tools">
                  <button class="btn btn-success" @click="newModal">Add New
                      <i class="fas fa-user-plus fa-fw"></i>
                  </button>
              </div>
            </div>
            <!-- /.box-header -->
            <div class="box-body table-responsive no-padding">
              <table class="table table-hover">
                <tbody><tr>
                  <th>ID</th>
                  <th>User</th>
                  <th>Email</th>
                  <th>Type</th>
                  <th>Registrered at</th>
                  <th>Modify</th>
                </tr>
                <tr v-for="user in users" :key="user.id">
                  <td>{{ user.id }}</td>
                  <td>{{ user.name }}</td>
                  <td>{{ user.email}}</td>
                  <td>{{ user.type | upText }}</td>
                  <td>{{ user.created_at | myDate }}</td>
                  <td>
                      <a href="" @click.prevent="editModal(user)">
                          <i class="fa fa-edit"></i>
                      </a>
                      /
                      <a href="" @click.prevent="deleteUser(user.id)">
                          <i class="fa fa-trash"></i>
                      </a>
                  </td>
                </tr>
              </tbody></table>
            </div>
            <!-- /.box-body -->
          </div>
          <!-- /.box -->
        </div>
      </div>

      <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="editmode" id="addNewLabel">Update User Info</h5>
                <h5 class="modal-title" v-show="!editmode" id="addNewLabel">Add New</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>

            <form @submit.prevent="editmode ? updateUser() : createUser()">
            <div class="modal-body">
                
              <div class="form-group">
                <input v-model="form.name" type="text" name="name" placeholder="Name"
                  class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                <has-error :form="form" field="name"></has-error>
              </div>

              <div class="form-group">
                <input v-model="form.email" type="text" name="email" placeholder="Email Adress"
                  class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                <has-error :form="form" field="email"></has-error>
              </div>

              <div class="form-group">
                <textarea v-model="form.bio" name="bio" id="bio" placeholder="Short bio for user (optional)"
                  class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                <has-error :form="form" field="bio"></has-error>
              </div>

              <div class="form-group">
                <select v-model="form.type" name="type" id="type"
                  class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                  <option value="">Select a user role</option>  
                  <option value="admin">Admin</option>  
                  <option value="user">Standard User</option>  
                  <option value="author">Author</option>  
                </select>
                <has-error :form="form" field="type"></has-error>
              </div>

              <div class="form-group">
                <input v-model="form.password" type="password" name="password" placeholder="Password" id="password"
                  class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                <has-error :form="form" field="password"></has-error>
              </div>
              
            </div>
              <div class="modal-footer">
                  <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                  <button type="submit" v-show="editmode" class="btn btn-success">Edit</button>
                  <button type="submit" v-show="!editmode" class="btn btn-primary">Create</button>
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
          users: {},
          form: new Form ({
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
        updateUser() {
          this.$Progress.start();
          this.form.put('api/user/'+this.form.id)
          .then(() => {
            toast.fire({
                type: 'success',
                title: 'User Updated successfully'
              })   
            this.$Progress.finish();
            $('#addNew').modal('hide'); 
            Fire.$emit('AfterCreate');
            }).catch(() => {
              this.$Progress.fail();
            }) 
          },
        editModal(user) {
          this.editmode = true;
          this.form.reset();
          $('#addNew').modal('show');
          this.form.fill(user);
        },
        newModal() {
          this.editmode = false;
          this.form.reset();
          $('#addNew').modal('show');
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

            if (result.value) {

              this.form.delete('api/user/'+id).then(() => {
                swal.fire(
                  'Deleted!',
                  'Your file has been deleted.',
                  'success'
                )
              Fire.$emit('AfterCreate');
              }).catch(() => {

              })
            }
          })
        },

        loadUsers() {
          axios.get('api/user').then(({ data }) => (this.users = data.data));
        },

        createUser() {
          this.$Progress.start();
          this.form.post('api/user')
          .then(()=>{
            Fire.$emit('AfterCreate');
            $('#addNew').modal('hide');
            toast.fire({
                type: 'success',
                title: 'User Created successfully'
              })       
            this.$Progress.finish();           
          })
          .catch(()=>{
            this.$Progress.fail();
          });
          
        }

      },

      mounted() {
          this.loadUsers();
          Fire.$on('AfterCreate',() => {
            this.loadUsers();
          });
          // setInterval(() => this.loadUsers(), 3000);
      } 
    }
</script>
