<template>
  <v-container fill-height fluid grid-list-xl>
    <v-layout justify-center wrap>
      <v-flex md12 style="margin-bottom: 48px">
        <material-card color="green" flat full-width title="User Table" text="list of all Users">
          <v-spacer></v-spacer>
          <v-text-field v-model="search" append-icon="search" label="Search" single-line hide-details> -->
           </v-text-field>
          <v-dialog v-model="ProxyDialog" max-width="500px" >
            <template v-slot:activator="{ on }">
              <v-btn color="general" dark class="mb-2" v-on="on" @click="proxyDialogLoad">Add new</v-btn>
            </template>

            <v-card >
              <v-card-text >
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="proxyItem.name" label="Name" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="proxyItem.remote_url" label="Remote URL" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-select  label="Choose User Credential" :items="proxyItem.UserList" item-text="username" item-value="_id" v-on:change="onUserChangeInProxy"></v-select>
                    </v-flex>
                    <v-flex xs12 sm6 md12 v-if="showCredentialBox">
                      <v-select  label="Choose Authorization Type" :items="proxyItem.dialogCredentialList" item-text="type" item-value="_id" v-model="proxyItem.SelectedCredential"></v-select>
                    </v-flex>
                     <v-flex xs12 sm12 md12>
                      <v-select box  attach multiple label="Policy" :items="AllowedPolicyList" item-text="name" item-value="name" v-model="proxyItem.policyName" v-on:change="onPolicyChangeInProxy"></v-select>
                    </v-flex>
                    <v-flex xs12>
                      <v-textarea
                        outline
                        name="input-7-4"
                        label="Add Policies"
                        v-model="proxyItem.policyMatrix"
                      ></v-textarea>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer />
                <v-btn color="blue darken-1" flat @click="closeProxyDialog" >Cancel</v-btn>
                <v-btn color="blue darken-1" flat @click="saveProxy" >Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <v-data-table :headers="userHeaders" :items="userList">
            <template slot="headerCell" slot-scope="{ header }">
              <span class="subheading font-weight-light text--darken-3" v-text="header.text" />
            </template>
            <template slot="items" slot-scope="{ item }">
              <td v-for="(k, i) in userHeaders" :key="i">{{ item[k.value] }}</td>
              <td class="text-xs-right">
                <v-icon medium class="mr-2" @click="getRoutesByProxyId(item._id,item.name)">mdi-arrow-down-drop-circle-outline</v-icon>
                <v-icon medium @click="editProxyItem(item,proxyDialogLoad)">edit</v-icon>
              </td>
            </template>
          </v-data-table>
        </material-card>
      </v-flex>
      <v-flex md12 v-if="selectedProxyItem.value" style="margin-bottom: 48px">
        <material-card color="blue" flat full-width :title="'Route List: '+ selectedProxyItem.name" text="list of Routes">
          <v-dialog v-model="RouteDialog" max-width="500px" >
            <template v-slot:activator="{ on }">
              <v-btn color="general" dark class="mb-2" v-on="on" @click="getPolicyList">Add new</v-btn>
            </template>

            <v-card>
              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12 sm6 md4>
                      <v-text-field v-model="editedRouteItem.name" label="*Name" />
                    </v-flex>
                    <v-flex xs12 sm6 md4>
                      <v-text-field v-model="editedRouteItem.base_path" label="*Base Path" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedRouteItem.remote_path" label="*Remote Path" />
                    </v-flex>
                    <v-flex xs12 sm6 md6>
                      <v-text-field v-model="editedRouteItem.method" label="*Method" />
                    </v-flex>
                     <v-flex xs12 sm12 md12>
                      <v-select box  attach multiple label="Policy" :items="AllowedPolicyList" item-text="name" item-value="name" v-model="editedRouteItem.policyName" v-on:change="onPolicyChangeInRoute"></v-select>
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
                <v-btn color="blue darken-1" flat @click="closeRouteDialog" >Cancel</v-btn>
                <v-btn color="blue darken-1" flat @click="saveRouteDialog" >Save</v-btn>
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
                <v-icon medium class="mr-2" @click="editRouteItem(item,getPolicyList)">edit</v-icon>
                <v-icon medium @click="deleteRouteItem(item)">delete</v-icon>
              </td>
            </template>
          </v-data-table>
        </material-card>
      </v-flex>
    </v-layout>
  </v-container>

</template>

<script>
  export default {
    /**
     * created();
     * called when page is created. load all the proxy setups
     * 
     */
    created() {
      this.$http.get('/users/query?field={}')
        .then(response => {
            
            this.userHeaders = Object.keys(response.data[0]).map(val => {
                return {
                    sortable: false,
                    text: val,
                    value: val
                }
            });
            this.userList = response.data;
        })
        .catch(error => console.log(error))
    },
    methods: {
      /**
       * onPolicyChangeInProxy();
       * when policy is changed inside dialog form for proxy list;
       * compute the policy matrix
       */
      onPolicyChangeInProxy(selectedPolicyName) {
        let tempPolicyItem = [];
        tempPolicyItem =  (this.proxyItem.policyMatrix === undefined) ? [] :  JSON.parse(this.proxyItem.policyMatrix);
        tempPolicyItem = this._PolicyChangeComputation(selectedPolicyName, tempPolicyItem);
        this.proxyItem.policyMatrix = JSON.stringify(tempPolicyItem, null, '\t');
      },
      /**
       * onPolicyChangeInRoute();
       * when policy is changed inside dialog form from route list;
       * compute the policy matrix
       */
      onPolicyChangeInRoute(selectedPolicyName) {
        let tempPolicyItem = [];
        tempPolicyItem = (this.editedRouteItem.policyMatrix === undefined ) ? [] :  JSON.parse(this.editedRouteItem.policyMatrix);
        tempPolicyItem = this._PolicyChangeComputation(selectedPolicyName, tempPolicyItem);
        this.editedRouteItem.policyMatrix = JSON.stringify(tempPolicyItem, null, '\t');
      },
      /**
       * _PolicyChangeComputation(selectedPolicyName, tempPolicyItem)
       * check if the policy matrix is valid and remove or add template if policy is selected
       */
      _PolicyChangeComputation(selectedPolicyName, tempPolicyItem){
        selectedPolicyName.forEach(policy=> {
          const ispolicyexist = tempPolicyItem.find(elem => elem.name === policy);
          if(!ispolicyexist) {
            tempPolicyItem.push(this.AllowedPolicyList.find(elem => elem.name=== policy));
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
      /**
       * onUserChangeInProxy(selectedUserId);
       * Load the allowed credential if user is changed in Proxy list
       */
      onUserChangeInProxy (selectedUserId) {
        this.$http.get(`/credentials/query?filter={"userid":"${selectedUserId}"}&field={}`).then(response => {
          this.proxyItem.dialogCredentialList = response.data;
          this.showCredentialBox = true;
        })
        .catch( error => console.log(error));
      },
      /**
       * proxyDialogLoad();
       * load users if the proxy dialog is loaded
       */
      proxyDialogLoad(){
        //get the list of users
        this.$http.get(`/users/query?field={"username":1}`).then(response => {
          this.proxyItem.UserList = response.data;
        })
        .catch(error => console.log(error));
        this.getPolicyList();
      },
      /**
       * getPolicyList()
       * load the list of allowed policy
       */
      getPolicyList() {
         this.$http.get(`/credentials/policy/list`).then( response => {
          console.log('policy',response)
          this.AllowedPolicyList = response.data
        })
        .catch( error => console.log(error));
      },
      /**
       * editProxyItem(item, triggerDialog);
       * when Item is selected from proxy list. load data to proxy dialog and trigger proxy load function
       */
      editProxyItem(item, triggerDialog) {
        this.proxyItem= {
          _id : item._id,
          SelectedCredential : item.credential,
          name: item.name,
          remote_url: item.remote_url,
          policyMatrix: JSON.stringify(item.policy,null, '\t'),
          state : 'edit'
        }
        // trigger the form intial function 
        triggerDialog();
        // update the credential value inside dialog form
        this.$http.get(`/credentials/query?filter={"_id": "${item.credential}"}`).then( response => {
          this.proxyItem.dialogCredentialList = response.data;
          this.showCredentialBox = true;
          this.ProxyDialog = true;
        })
      },
      /**
       * editRouteItem(item, triggerDialog);
       * when Item is selected from Route list. load data to route dialog and trigger route dialog load function
       */
      editRouteItem (item, triggerDialog) {
        console.log(item);
        this.RouteDialog = true;
        this.editedRouteItem = {
          _id: item._id,
          name: item.name,
          base_path: item.base_path,
          remote_path: item.remote_path,
          method: item.method,
          policyMatrix: JSON.stringify(item.policy,null, '\t'),
          state : 'edit'
        }
        // load policy options
        triggerDialog();
      },
      /**
       * getRoutesByProxyId( id, name);
       * load the route related to proxy selected. 
       */
      getRoutesByProxyId(id,name) {
        this.selectedProxyItem.value= id;
        this.selectedProxyItem.name = name
        this.$http.get(`/routes/query?filter={"proxyId":"${id}"}`).then(response => {
          this.RouteList = response.data;
          this.RouteHeader = this._generateHeader(Object.keys(this.RouteList[0]));
          console.log(this.RouteHeader, this.RouteList)
          this.selectedProxyItem.state = true;
        }).catch(error => console.log(error));
        console.log(id);
      },
      _generateHeader(listOfHeader) {
        const ignoreHeader = ['__v','proxyId', '_id','creation_date'];
        const list =  listOfHeader.map(element => {
          return {
            sortable: false,
            text: element,
            value: element
          }
        }).filter(elem => ignoreHeader.indexOf(elem.value) === -1);
        list.push({
          sortable: false,
          text: 'Action',
          value: 'Action'
        });
        return list;
        
      },
      closeRouteDialog () {
        this.RouteDialog = false;
        this.editedRouteItem = {}
      },
      closeProxyDialog () {
        this.ProxyDialog = false
        setTimeout(() => {
          this.proxyItem = Object.assign({}, this.defaultItem)
          this.showCredentialBox = false;
          this.editedIndex = 1
          //this.proxyItem = {}
        }, 300)
      },
      saveRouteDialog () {
        let url = '';
        const data = {
          name: this.editedRouteItem.name,
          base_path : this.editedRouteItem.base_path,
          remote_path: this.editedRouteItem.remote_path,
          method: this.editedRouteItem.method,
          policy : JSON.parse(this.editedRouteItem.policyMatrix || '[]'),
        }
        if( this.editedRouteItem.state === 'edit'){
          url = 'routes/update/'+this.editedRouteItem._id
        } else {
          url = 'routes/create';
          data.proxyId = this.selectedProxyItem.value;
        }
        this.$http.post(url, data ).then( response => {
          this.editedRouteItem = {};
          // update list again 
          this.getRoutesByProxyId(this.selectedProxyItem.value, this.selectedProxyItem.name);
          this.RouteDialog = false
        }).catch(error => console.log(error));
      },
      deleteRouteItem(item){
        this.$http.delete(`/routes/delete/${item._id}`).then(response => {
            this.getRoutesByProxyId(this.selectedProxyItem.value, this.selectedProxyItem.name);
        }).catch( error => console.log(error));
      },
      saveProxy () {
        let url = '';
        const data = {
          name: this.proxyItem.name,
          policy: JSON.parse(this.proxyItem.policyMatrix),
          remote_url: this.proxyItem.remote_url,
          credential: this.proxyItem.SelectedCredential,
        }
        if (this.proxyItem.state === 'edit') {
          url = '/proxy/update/'+this.proxyItem._id
        } else {
          url  = 'proxy/create';
        }
        const self = this;
        this.$http.post(url, data).then(response => {
          this.proxyItem = Object.assign({}, this.defaultItem)
          this.showCredentialBox = false;
          // replace if id entry is updated
          const elem = this.userList.findIndex(cluster => cluster._id == response.data._id);
          if(elem >= 0 ){
            self.userList[elem] = Object.assign(self.userList[elem], response.data);
          } else {
            self.userList.push(response.data);
          }
          self.ProxyDialog = false
        }).catch(error =>  {
          console.log(error)
        });
      }
    },
    data: () => ({
      ProxyDialog: false,
      RouteDialog: false,
      search : '',
      userHeaders: [ ],
      userList: [],
      selectedRoute: {
        value: '',
        state: false
      },
      selectedProxyItem: {
        value: '',
        state: false,
        name: ''
      },
      RouteList: [],
      RouteHeader: [],

      editedIndex: -1,
      proxyItem: {
        name: '',
        remote_url: '',
        SelectedCredential: '',
        policyName: '',
        policyMatrix: '[]',
        dialogCredentialList: [],
        UserList: [],
   
      },
      showCredentialBox : true,
      editedRouteItem: {
        policyMatrix: '[]',
        base_path: '',
        remote_path: '',
        method: '',
      },
      defaultItem: { 
        name: '',
        remote_url: '',
        SelectedCredential: '',
        policyName: '',
        policyMatrix: '[]',
        dialogCredentialList: [],     
        UserList: [],

       },
      AllowedPolicyList: []
    }),
    watch: {
      ProxyDialog (val) {
        val || this.closeProxyDialog()
      },
      RouteDialog (val) {
        val || this.closeRouteDialog();
      }
    },
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },
  }
</script>
