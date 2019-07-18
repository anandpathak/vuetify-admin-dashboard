<template>
  <v-container fill-height fluid grid-list-xl>
    <v-layout justify-center wrap>
      <v-flex md12>
        <material-card color="green" flat full-width title="Route Cluster" text="list of all Route cluster">
          <v-spacer></v-spacer>
          <v-text-field v-model="search" append-icon="search" label="Search" single-line hide-details> -->
           </v-text-field>
          <v-dialog v-model="dialog" max-width="500px" >
            <template v-slot:activator="{ on }">
              <v-btn color="general" dark class="mb-2" v-on="on" @click="addNew">Add new</v-btn>
            </template>

            <v-card>
              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedItem.name" label="Name" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedItem.remote_url" label="Remote URL" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-select  label="Choose User Credential" :items="dialogUserList" item-text="username" item-value="_id" v-on:change="onUserChange"></v-select>
                    </v-flex>
                    <v-flex xs12 sm6 md12 v-if="dialogCredentialList.length > 0 ">
                      <v-select  label="Choose Authorization Type" :items="dialogCredentialList" item-text="type" item-value="_id" v-model="editedItem.SelectedCredential"></v-select>
                    </v-flex>
                     <v-flex xs12 sm12 md12>
                      <v-select box  attach multiple label="Policy" :items="dialogPolicyList" item-text="name" item-value="name" v-model="editedItem.policyName" v-on:change="onPolicyChange"></v-select>
                    </v-flex>
                    <v-flex xs12>
                      <v-textarea
                        outline
                        name="input-7-4"
                        label="Add Policies"
                        v-model="editedItem.policyMatrix"
                      ></v-textarea>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" flat @click="close" >Cancel</v-btn>
                <v-btn color="blue darken-1" flat @click="save" >Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-data-table :headers="headers" :items="RouteCluster">
            <template slot="headerCell" slot-scope="{ header }">
              <span class="subheading font-weight-light text--darken-3" v-text="header.text" />
            </template>
            <template slot="items" slot-scope="{ item }">
              <td>{{ item.name }}</td>
              <td>{{ item.remote_url }}</td>
              <td>{{ item._id }}</td>
              <td class="text-xs-right">
                <v-icon medium class="mr-2" @click="getItems(item._id)">mdi-arrow-down-drop-circle-outline</v-icon>
                <v-icon medium @click="editItem(item._id)">edit</v-icon>
              </td>
            </template>
          </v-data-table>
        </material-card>
      </v-flex>
      <v-flex md12 v-if="selectedRoute.value">
        <material-card color="blue" flat full-width title="Route List" text="list of Routes">
          <v-dialog v-model="RouteDialog" max-width="500px" >
            <template v-slot:activator="{ on }">
              <v-btn color="general" dark class="mb-2" v-on="on" @click="addRouteIntoCluster">Add new</v-btn>
            </template>

            <v-card>
              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedRouteItem.base_path" label="Base Path" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedRouteItem.remote_path" label="Remote Path" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedRouteItem.method" label="Method" />
                    </v-flex>
                     <v-flex xs12 sm12 md12>
                      <v-select box  attach multiple label="Policy" :items="dialogPolicyList" item-text="name" item-value="name" v-model="editedRouteItem.policyName" v-on:change="onPolicyChangeEachRoute"></v-select>
                    </v-flex>
                    <v-flex xs12>
                      <v-textarea
                        outline
                        name="input-7-4"
                        label="Add Policies"
                        v-model="editedRouteItem.policyMatrix"
                      ></v-textarea>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" flat @click="closeRouteModel" >Cancel</v-btn>
                <v-btn color="blue darken-1" flat @click="saveRouteModel" >Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-data-table :headers="RouteHeader" :items="RouteList">
            <template slot="headerCell" slot-scope="{ header }">
              <span class="subheading font-weight-light text--darken-3" v-text="header.text" />
            </template>
            <template slot="items" slot-scope="{ item }">

              <td v-for="(k, i) in RouteHeader" :key="i">{{ item[k.value] }}</td>
              <td class="text-xs-right">
                <v-icon medium class="mr-2" @click="editItem(props.item)">edit</v-icon>
                <v-icon medium @click="deleteItem(props.item)">delete</v-icon>
              </td>
            </template>
          </v-data-table>
        </material-card>
      </v-flex>
    </v-layout>
  </v-container>

</template>

<script>
import { constants, pbkdf2 } from 'crypto';
  export default {
    created() {
      this.$http.get('/routes/query?field={"remote_url":1,"name":1}')
        .then(response => {
          this.RouteCluster = response.data
        })
        .catch(error => console.log(error))
    },
    methods: {
      onPolicyChange(selectedPolicyName) {
        let tempPolicyItem = [];
        tempPolicyItem =  (this.editedItem.policyMatrix === undefined || this.editedItem.policyMatrix == '') ? [] :  JSON.parse(this.editedItem.policyMatrix);
        tempPolicyItem = this._PolicyChangeComputation(selectedPolicyName, tempPolicyItem);
        this.editedItem.policyMatrix = JSON.stringify(tempPolicyItem, null, '\t');
      },
      onPolicyChangeEachRoute(selectedPolicyName) {
        let tempPolicyItem = [];
        tempPolicyItem = (this.editedRouteItem.policyMatrix === undefined || this.editedRouteItem.policyMatrix == '') ? [] :  JSON.parse(this.editedRouteItem.policyMatrix);
        tempPolicyItem = this._PolicyChangeComputation(selectedPolicyName, tempPolicyItem);
        this.editedRouteItem.policyMatrix = JSON.stringify(tempPolicyItem, null, '\t');
      },
      _PolicyChangeComputation(selectedPolicyName, tempPolicyItem){
        selectedPolicyName.forEach(policy=> {
          const ispolicyexist = tempPolicyItem.find(elem => elem.name === policy);
          if(!ispolicyexist) {
            tempPolicyItem.push(this.dialogPolicyList.find(elem => elem.name=== policy));
          }
        });
        tempPolicyItem =  tempPolicyItem.filter(element => {
          const unselectedPoloicy = selectedPolicyName.find(elem => elem === element.name)
          if(!unselectedPoloicy){
            return false
          } 
          return true;
        });
        return tempPolicyItem;
      },
      onUserChange (selectedUserId) {
        this.$http.get(`/credentials/query?filter={"userid":"${selectedUserId}"}&field={}`).then(response => {
          this.dialogCredentialList = response.data;
        })
        .catch( error => console.log(error));
      },
      addNew(){
        this.$http.get(`/users/query?field={"username":1}`).then(response => {
          this.dialogUserList = response.data;
        })
        .catch(error => console.log(error));
        this.addRouteIntoCluster();
      },
      addRouteIntoCluster() {
         this.$http.get(`/credentials/policy/list`).then( response => {
          console.log('policy',response)
          this.dialogPolicyList = response.data
        })
        .catch( error => console.log(error));
      },
      getItems(id) {
        this.selectedRoute.value= id;
        this.$http.get(`/routes/query?filter={"_id":"${id}"}&field={"routes":1}`).then(response => {
          this.RouteList = response.data[0].routes;
          this.RouteHeader = this._generateHeader(Object.keys(this.RouteList[0]));
          console.log(this.RouteHeader, this.RouteList)
          this.selectedRoute.state = true;
        }).catch(error => console.log(error));
        console.log(id);
      },
      _generateHeader(listOfHeader) {
        return listOfHeader.map(element => {
          return {
            sortable: false,
            text: element,
            value: element
          }
        });
      },
      closeRouteModel () {
        this.RouteDialog = false;
      },
      close () {
        this.dialog = false
        setTimeout(() => {
          // this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = 1
        }, 300)
      },
      saveRouteModel () {
        const data = {
          base_path : this.editedRouteItem.base_path,
          remote_path: this.editedRouteItem.remote_path,
          method: this.editedRouteItem.method,
          policy : JSON.parse(this.editedRouteItem.policyMatrix || '[]') 
        }
        this.$http.post('routes/appendRoute/'+this.selectedRoute.value, data ).then( response => {
          this.editedRouteItem = {};
          this.RouteList.push(data);
          this.RouteDialog = false
        }).catch(error => console.log(error));
      },
      save () {
        console.log(this.editedItem);
        const data = {
          name: this.editedItem.name,
          policy: JSON.parse(this.editedItem.policyMatrix),
          remote_url: this.editedItem.remote_url,
          credential: this.editedItem.SelectedCredential,
          routes: [],
        }
        this.$http.post('/routes/add', data).then(response => {
          console.log(response.data);
          this.editedItem = {};
          this.RouteCluster.push(response.data);
          this.dialog = false
        }).catch(error =>  {
          console.log(error)
        });
      }
    },
    data: () => ({
      dialog: false,
      RouteDialog: false,
      search : '',
      headers: [{
          sortable: false,
          text: 'Name',
          value: 'name'
        },
        {
          sortable: false,
          text: 'Remote URL',
          value: 'remote_url'
        },
        {
          sortable: false,
          text: 'ID',
          value: '_id'
        },
        {
          sortable: false,
          text: 'Action',
          value: 'action',
          align: 'right'
        }
      ],
      RouteCluster: [],
      selectedRoute: {
        value: '',
        state: false
      },
      RouteList: [],
      RouteHeader: [],

      editedIndex: -1,
      editedItem: {
        name: '',
        remote_url: '',
        SelectedCredential: '',
        policyName: '',
        policyMatrix: ''      
      },
      editedRouteItem: {
        policyMatrix: '',
        base_path: '',
        remote_path: '',
        method: ''

      },
      defaultItem: { },
      dialogUserList: [],
      dialogCredentialList: [],
      dialogPolicyList: []
    }),
    watch: {
      dialog (val) {
        val || this.close()
      },
      RouteDialog (val) {
        val || this.close();
      }
    },
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },
  }
</script>