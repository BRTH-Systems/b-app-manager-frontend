<template>
  <v-app id="inspire">
    <v-app-bar
      app
      color="white"
      flat
    >
      <v-container class="py-0 fill-height">
        <v-avatar
          class="mr-10"
          color="grey darken-1"
          size="32"
        ></v-avatar>

        <v-btn
          v-for="link in headMenuItems"
          :key="link"
          text
        >
          {{ link }}
        </v-btn>

        <v-spacer></v-spacer>

        <!-- <v-responsive max-width="260">
          <v-text-field
            dense
            flat
            hide-details
            rounded
            solo-inverted
          ></v-text-field>
        </v-responsive> -->
      </v-container>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <v-row>
          <v-col cols="3">
            <v-sheet rounded="lg">
              <v-list color="transparent">

                <v-list-item
                  color="grey lighten-4"
                >
                  <v-list-item-content>
                    <v-list-item-title>
                      Apps

                      <v-dialog v-model="newApp.dialogOpened" max-width="1000px">
                            <template v-slot:activator="{ on }">
                                <v-btn
                                  v-on="on"
                                  style="margin: 1rem"
                                  fab
                                  size: x-small
                                >
                                  <v-icon>mdi-plus</v-icon>
                                </v-btn>
                            </template>
                            <v-card>
                                <v-card-title>
                                    <span class="headline">New App</span>
                                </v-card-title>
                                <v-card-text>
                                    <v-form>
                                    <v-container>
                                      <v-row>
                                            <v-col>
                                              <v-text-field
                                                  v-model="newApp.name"
                                                  label="App name"
                                                  clearable
                                              />
                                            </v-col>
                                            <v-col>
                                              <template>
                                                <v-combobox
                                                  v-model="newApp.deployments.selected"
                                                  :items="newApp.deployments.available"
                                                  chips
                                                  clearable
                                                  label="Deployments in app"
                                                  multiple
                                                  solo
                                                >
                                                  <template v-slot:selection="{ attrs, item, selected }">
                                                    <v-chip
                                                      v-bind="attrs"
                                                      :input-value="selected"
                                                      close
                                                      @click="editSelectedDeployment(item)"
                                                      @click:close="removeSelectedDeployment(item)"
                                                    >
                                                      <strong>{{ item }}</strong>
                                                    </v-chip>
                                                  </template>
                                                </v-combobox>
                                              </template>
                                            </v-col>
                                      </v-row>
                                    </v-container>
                                    </v-form>
                                </v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn color="blue darken-1" text @click="newApp.dialogOpened = false">Mégse</v-btn>
                                    <v-btn color="blue darken-1" text @click="createApp(); newApp.dialogOpened = false">Létrehozás</v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>

                      <v-btn
                        style="margin: 1rem"
                        fab
                        size: x-small
                        :loading="loading.apps"
                        @click="getClusterState()"
                      >
                        <v-icon>mdi-refresh</v-icon>
                      </v-btn>
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-divider class="my-2"></v-divider>

                <v-list-item
                  v-for="app in apps"
                  :key="app.name"
                  link
                  :disabled="app.name === selectedAppName"
                  @click="newAppSelected(app)"
                >
                  <v-list-item-content>
                      {{ app.name }}
                    <v-list-item-title>
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

              </v-list>
            </v-sheet>
          </v-col>

          <v-col>
            <v-sheet
              min-height="70vh"
              rounded="lg"
            >
              <template>
                <v-container>
                  <v-row>
                    <v-col>
                      {{ selectedAppName }}
                      <v-btn
                        v-if="selectedAppName.length > 0"
                        style="margin: 1rem"
                        fab
                        size: x-small
                        @click="deleteApp(selectedAppName)"
                      >
                        <v-icon>mdi-trash-can-outline</v-icon>
                      </v-btn>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col
                      v-for="deployment in getSelectedDeployments()"
                      :key="deployment.name"
                      cols="4"
                    >
                      <v-card height="300">
                        <v-card-title>
                          {{ deployment.name }}
                        </v-card-title>
                        <v-card-text>
                          {{ deployment.pods.length }} pod / {{ deployment.pods.filter(x => x.status != 'Running').length }} unhealthy
                        </v-card-text>
                      </v-card>
                    </v-col>
                  </v-row>
                </v-container>
              </template>
            </v-sheet>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
  export default {
    data: () => ({
      headMenuItems: [
        'Apps',
      ],
      apps: [],
      selectedAppName: '',
      loading: {
        apps: false,
      },
      newApp: {
        dialogOpened : false,
        name: '',
        deployments: {
          available: ['mariadb', 'wordpress'],
          selected: []
        },
      }
    }),

    methods: {
      removeSelectedDeployment(item){
        this.newApp.deployments.selected.splice(this.newApp.deployments.selected.indexOf(item), 1)
        this.newApp.deployments.selected = [...this.newApp.deployments.selected]
      },
      editSelectedDeployment(item){
        console.log(item)
      },
      newAppSelected(app){
        this.selectedAppName = app.name
      },
      async getClusterState(){
        this.loading.apps = true

        var requestOptions = {
            method: 'GET',
            credentials: 'include',
            redirect: 'follow',
          }

          try{
            const res = await fetch( 'http://172.17.0.2/clusterstate', requestOptions)
            const resText = await res.text()

            if(res.status != 200){
              console.log({"status": res.status, "msg": resText})
              this.loading.apps = false
              return false
            }

            const responseObject = JSON.parse(resText)
            this.apps = responseObject.namespaces

          }catch(e){
            console.log(e)
            this.logInErrorMsg = e
            this.loading.apps = false
            return false
          }

          this.loading.apps = false
          return true
      },
      getSelectedDeployments(){
        let selectedApp = this.apps.find(x => x.name === this.selectedAppName)
        if(selectedApp != undefined) return selectedApp.deployments
        else return []
      },
      async deleteApp(appName){
        var requestOptions = {
            method: 'DELETE',
            credentials: 'include',
            redirect: 'follow',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({
              appName: appName
            })
          }

          try{
            const res = await fetch( 'http://172.17.0.2/manageapps', requestOptions)
            const resText = await res.text()

            if(res.status != 200){
              console.log({"status": res.status, "msg": resText})
              return false
            }

            this.getClusterState()

          }catch(e){
            console.log(e)
            this.logInErrorMsg = e
            return false
          }

          return true
      },
      async createApp(){

        if(this.newApp.name.length < 1) return false
        if(this.newApp.deployments.length < 1) return false

        let newAppObject = {
            "appName": this.newApp.name,
            "appDeployments": []
        }

        this.newApp.deployments.selected.forEach((deploymentName) => {
          let deploymentSchema = {
            name: deploymentName,
            image: deploymentName + ':latest'
          }
          newAppObject.appDeployments.push(deploymentSchema)
        })

        var requestOptions = {
            method: 'POST',
            credentials: 'include',
            redirect: 'follow',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(newAppObject)
          }

          try{
            const res = await fetch( 'http://172.17.0.2/manageapps', requestOptions)
            const resText = await res.text()

            if(res.status != 200){
              console.log({"status": res.status, "msg": resText})
              return false
            }

            this.getClusterState()

          }catch(e){
            console.log(e)
            this.logInErrorMsg = e
            return false
          }

          this.newApp.name = ''
          this.newApp.deployments = []
          return true
      }
    }
  }
</script>