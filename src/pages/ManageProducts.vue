<template>
    <q-page class="flex flex-center bg-grey-1">
        <div>
            <q-splitter
                v-model="splitterModel"
                style="height: 95vh; width: 100vw;"
            >
            <template v-slot:before>
        
                <q-tabs
                v-model="tab"
                vertical
                class="text-grey-8 bg-white full-height relative-position"
                active-color="black"
                active-bg-color="amber-3"
                >
                    <q-scroll-area class="fit">
                        <div class="text-h6 text-center q-py-md">Products & Services</div>
                        <q-tab name="SHOP" icon="shopping_cart" label="SHOP" />
                        <q-tab name="GARAGE" icon="build" label="GARAGE"/>
                        <q-tab name="SERVICES"  label="SERVICES" icon="local_car_wash"/>

                        <div class="text-h6 text-center q-py-md">File Maintenance</div>
                        <q-tab name="BRAND"  label="BRANDS" />
                        <q-tab name="PARTS"  label="PARTS" ></q-tab>
                        <q-tab name="CAR MODELS"  label="CAR MODELS" ></q-tab>
                        
                    </q-scroll-area>
                </q-tabs>
        
            </template>
            <template v-slot:after>
                <div class="q-px-md text-grey-8">
                    <q-table
                        :title="getTitle"
                        :data="filterFileMaintenance"
                        :columns="getColumns"
                        row-key=".key"
                        :filter="filter"
                        :grid="viewChange"
                        :grid-header="viewChange"
                        :pagination.sync="pagination"
                        class="shadow-0 bg-grey-1 text-grey-8"
                    >
                        <template v-slot:top-right>
                            <q-btn-toggle
                            dense
                            flat
                            rounded
                            class="q-mx-md"
                            v-model="tableView"
                            toggle-color="amber"
                            toggle-text-color="amber"
                            :options="[
                                {value: 'list',slot: 'list'},
                                {value: 'grid',slot: 'grid'},
                            ]"
                            >
                                <template v-slot:list>
                                    <q-icon right name="list" class="q-px-md"/>
                                </template>
                                <template v-slot:grid>
                                    <q-icon right name="grid_on" class="q-px-md"/>
                                </template>                            
                            </q-btn-toggle>

                            <q-input color="grey-8" borderless dense debounce="300" v-model="filter" placeholder="Search" clearable>
                                <template v-slot:prepend>
                                    <q-icon name="search" />
                                </template>
                            </q-input>

                        </template>

                        <template v-slot:item="props" v-show="viewChange">
                            <div class="q-pa-sm col-xs-12 col-sm-6 col-md-3 col-lg-2 grid-style-transition">
                            <q-card  class="q-pa-sm" >
                            
                                <q-img
                                    :src="props.row.photo"
                                    basic
                                    :ratio="1"
                                    :contain="tab != 'BRAND'"
                                />

                                <q-card-section>
                                    <div class="text-h6">{{props.row.name}}  <q-chip class="q-mt-sm" size="sm" v-show="props.row.type == 'OTHERS' && props.row.type">{{props.row.type}}</q-chip></div>
                                    <div class="text-subtitle" v-show="props.row.shortDesc != ''">{{props.row.shortDesc}}</div>
                                    <div class="text-subtitle text-white" v-show="props.row.shortDesc == ''"> - </div>
                                    <q-chip class="q-mt-sm" size="sm" v-for="(year,i) in props.row.year" :key="i" v-show="props.row.year">{{year}}</q-chip>
                                </q-card-section>
                            </q-card>

                            </div>
                        </template>



                    </q-table>
                </div>
            </template>


            </q-splitter>
        </div>
        <q-page-sticky position="bottom-right" :offset="[18, 18]" >
            <q-btn fab icon="add" color="amber" class="text-black" @click="checkDialogOpen"/>
        </q-page-sticky>

        <q-dialog v-model="showDialog">
            <q-card style="width: 500px; max-width: 60vw;">
                <q-card-section class="row items-center q-pb-none">
                <div class="text-h6">ADD NEW {{tab}}</div>
                <q-space />
                <q-btn icon="close" flat round dense v-close-popup @click="newAddReset"/>
                </q-card-section>

                <q-card-section>
                    <div>
                       <q-select
                            outlined
                            v-model="newAdd.brand"
                            :options="brandOpt"
                            use-chips
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Brand Selection"
                            class="full-width q-mb-md"
                            v-show="tab == 'CAR MODELS'"
                            emit-value
                        />
                        <div class="container row" v-show="tab == 'BRAND'">
                            <q-input color="amber" outlined v-model="newAdd.name" :label="returnName.name" class="col q-mr-md" v-show="tab == 'BRAND'"/>
                            <q-select
                                outlined=""
                                v-model="newAdd.brandType"
                                :options="brandType"
                                stack-label
                                color="amber-9"
                                popup-content-class="text-black"
                                label="Brand Type"
                                class="col-4"
                                v-show="tab == 'BRAND'"
                            />
                        </div>
                        <q-input color="amber" outlined v-model="newAdd.name" :label="returnName.name" v-show="tab != 'BRAND'"/>
                        <q-uploader
                            :url="newAdd.photo"
                            :hide-upload-btn="true"
                            :label="returnName.photo"
                            class="full-width q-mt-md"
                            color="amber"
                            name="brandPhoto"
                            ref="upldBrand"
                            :max-total-size="10000000"
                            accept="'.gif,.GIF,.jpg,.JPG,.jpeg,.JPEG,.png,.PNG'"
                            :multiple="false"
                            @added="photoAddedBrand"
                            @removed="photoRemovedBrand"
                        />
                        <q-input
                            :label="returnName.short"
                            v-model="newAdd.shortdesc"
                            outlined
                            type="text"
                            v-show="tab != 'BRAND'"
                            class="full-width q-mt-md"
                            color="amber"
                            autogrow
                        />
 
                        <q-select
                            outlined=""
                            v-model="newAdd.year"
                            multiple
                            :options="yearOptions"
                            use-chips
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Car Year Selection"
                            class="full-width q-mt-md"
                            v-show="tab != 'BRAND' && tab != 'PARTS'"
                        />

                    </div>

                </q-card-section>
                <q-card-actions align="right" class="q-pa-md">
                    <q-btn size="md" flat v-close-popup @click="newAddReset">CANCEL</q-btn>
                    <q-btn size="md" flat color="amber" @click="addNewRecord">ADD {{tab}}</q-btn>
                </q-card-actions>
                <q-inner-loading :showing="loadingDialog">
                    <q-spinner-hourglass size="100px" color="amber"></q-spinner-hourglass>
                </q-inner-loading>
            </q-card>
        </q-dialog>
        
        <q-dialog v-model="showShopDialog">
            <q-card style="width: 500px; max-width: 60vw;">
                <q-card-section class="row items-center q-pb-none">
                <div class="text-h6">ADD NEW {{tab}}</div>
                <q-space />
                <q-btn icon="close" flat round dense v-close-popup @click="newAddReset"/>
                </q-card-section>
                <q-card-section>
                    <q-input color="amber" outlined v-model="newProduct.name" label="Product Name"/>
                    <div class="container row">
                        <q-select
                            outlined=""
                            v-model="newProduct.brand"
                            :options="brandProductsOpt"
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Brand Type"
                            class="q-mt-md q-mr-md col"
                            emit-value
                        />
                        <q-select
                            outlined=""
                            v-model="newProduct.partType"
                            :options="partsOpt"
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Parts Type"
                            class="q-mt-md col"
                            emit-value
                        />
                    </div>
                        <q-select
                            outlined=""
                            v-model="newProduct.carModel"
                            :options="carModelOpt"
                            use-chips
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Car Model"
                            class="q-mt-md col-12"
                            emit-value
                        />
                        <q-select
                            outlined=""
                            v-model="newProduct.year"
                            multiple
                            :options="selectCarModelYears"
                            use-chips
                            stack-label
                            color="amber-9"
                            popup-content-class="text-black"
                            label="Car Year Selection"
                            class="full-width q-mt-md"
                        />
                        <q-input
                            label="Product Description"
                            v-model="newProduct.desc"
                            outlined
                            type="text"
                            class="full-width q-mt-md"
                            color="amber"
                            autogrow
                        />
                    <div class="container row q-mt-md">
                        <q-input color="amber" min="0" outlined v-model="newProduct.srp" label="SRP (If available)" class="col q-mr-md" type="number"/>
                        <q-input color="amber" min="0" outlined v-model="newProduct.sellPrice" label="Selling Price" class="col" type="number"/>
                    </div>
                        <q-uploader
                            :url="newProduct.uploadUrl"
                            :hide-upload-btn="true"
                            label="Product Gallery"
                            class="full-width q-mt-md shadow-1"
                            color="amber"
                            name="brandPhoto"
                            ref="upldGallery"
                            :multiple="true"
                            :max-total-size="10000000"
                            accept="'.gif,.GIF,.jpg,.JPG,.jpeg,.JPEG,.png,.PNG'"
                            @added="photoAddedGallery"
                            @removed="photoRemovedBrand"
                        >
                              <template v-slot:list="scope">
                                <q-list separator>

                                <q-item v-for="file in scope.files" :key="file.name">
                                    <q-item-section>
                                    <q-item-label class="full-width ellipsis">
                                        {{ file.name }}
                                    </q-item-label>
                                    </q-item-section>

                                    <q-item-section
                                    v-if="file.__img"
                                    thumbnail
                                    class="gt-xs"
                                    >
                                    <img :src="file.__img.src">
                                    </q-item-section>

                                    <q-item-section top side>
                                    <q-btn
                                        class="gt-xs"
                                        size="12px"
                                        flat
                                        dense
                                        round
                                        icon="delete"
                                        @click="scope.removeFile(file)"
                                    />
                                    </q-item-section>
                                </q-item>

                                </q-list>
                            </template>
                        </q-uploader>
                </q-card-section>
               
                <q-card-section>
                     <q-separator/>
                    <div class="q-pa-md">
                        <div class="text-center full-width q-mb-sm text-grey-8">INVENTORY</div>
                        <div class="container row">
                            <q-btn color="amber" outline class="col" @click="showInitialStocks = true" v-show="showInitialStocks == false">ADD INITIAL STOCKS</q-btn>
                            <q-input outlined dense type="number" label="Stocks" class="col-8" v-model="newStock.qty" v-show="showInitialStocks"></q-input>
                            <q-btn color="grey-8" flat class="col q-ml-md" @click="showInitialStocks = false" v-show="showInitialStocks == true">CANCEL</q-btn>
                        </div>
                    </div>
                    <q-separator class="q-mt-sm"/>
                </q-card-section>
                <q-card-actions align="right" class="q-pa-md">
                    <q-btn size="md" flat v-close-popup>CANCEL</q-btn>
                    <q-btn size="md" flat color="amber">ADD {{tab}}</q-btn>
                </q-card-actions>
                <q-inner-loading :showing="loadingDialog">
                    <q-spinner-hourglass size="100px" color="amber"></q-spinner-hourglass>
                </q-inner-loading>
            </q-card>
        </q-dialog>

    </q-page>
</template>
<style>
    
</style>
<script>
import axios from 'axios'
import { date } from 'quasar'
let sri = require('simple-random-id')

export default {
        name: 'ManageProducts',
        data () {
            return {
                carModels: [],
                brands: [],
                parts: [],
                splitterModel: 20,
                tab: 'SHOP',
                filter: '',
                tableView: 'grid',
                showDialog: false,
                showShopDialog: false,
                newAdd: {
                    name: '',
                    brandType: 'CAR BRAND',
                    photo: '',
                    shortdesc: '',
                    carModel: '',
                    year: [],
                    color: [],
                    brand: []
                },
                newProduct: {
                    name: '',
                    brand: '',
                    uploadUrl: '',
                    desc: '',
                    year: [],
                    images: [],
                    partType: '',
                    carModel: '',
                    sellPrice: 0,
                    srp: 0,
                },
                showInitialStocks: false,
                newStock: {
                    qty: 0,
                },
                uploadedPhoto: null,
                loadingDialog: false,
                pagination: {descending: false, page: 1, rowsPerPage: 10},
                yearOptions: ['2000','2001','2002','2003','2004','2005','2006','2007','2008','2009','2010','2011','2012','2013','2014','2015','2016','2017','2018','2019','2020'],
                carOptions: ['ANY'],
                brandType: ['CAR BRAND','OTHERS']
            }
        },
        mounted(){
            this.$binding('brands', this.$firestoreApp.collection('brands'))
                .then(brands => {
                    console.log(brands, 'brands - mounted')
                }),
            this.$binding('parts', this.$firestoreApp.collection('parts'))
                .then(parts => {
                    console.log(parts, 'parts - mounted')
                })
                ,
            this.$binding('carModels', this.$firestoreApp.collection('carModels'))
                .then(carModels => {
                    console.log(carModels, 'carModels - mounted')
                })
        },
        computed:{
            selectCarModelYears(){
                try {
                    let arr = []
                    let opt = this.$lodash.filter(this.carModels,m=> {
                        return m.name + ' ' + m.shortDesc == this.newProduct.carModel
                    })
                    console.log(opt,'year')
                    return opt[0].year
                } catch(err) {
                    return []
                }
            },
            carModelOpt(){
                let arr = []
                let opt = this.$lodash.map(this.carModels,m=> {
                    if(m.brand == this.newProduct.brand){
                        arr.push({label: m.name + ' ' + m.shortDesc,value: m.name + ' ' + m.shortDesc})
                    }
                })
                return arr
            },
            partsOpt(){
                let opt = this.parts.map(m=> {
                    return {
                        label: m.name,
                        value: m.name
                    }
                })
                return opt
            },
            brandProductsOpt(){
                let opt = this.brands.map(m=> {
                    return {
                        label: m.name,
                        value: m.name
                    }
                })
                return opt
            },
            brandOpt(){
                let arr = []
                let opt = this.$lodash.map(this.brands, m => {
                    if(m.type == 'CAR BRAND'){
                        arr.push({label: m.name,value: m.name}) 
                    } 
                })
                console.log(arr,'arr')
                return arr
            },
            filterFileMaintenance(){
                if(this.tab == 'BRAND'){
                    return this.brands
                } else if (this.tab == 'PARTS'){
                    return this.parts
                } else if (this.tab == 'CAR MODELS'){
                    return this.carModels
                } else {
                    // return [{name: ''}]
                }
            },
            getColumns(){
                let basic = [   
                    {
                    name: 'name',
                    required: true,
                    label: 'Name',
                    align: 'left',
                    sortable: true,
                    field: 'name'
                    },

                ]

                if(this.tab == 'PARTS' || this.tab == 'CAR MODELS'){
                    basic.push({name:'shortDesc',required: true,label: 'Short DESC',align: 'left', sortable: true,field: 'shortDesc'})
                    
                }

                basic.push({name: 'action',label: 'Actions',align: 'left'})
                return basic
            },
            getTitle(){
                return this.tab
            },
            viewChange(){
                if(this.tableView == 'list'){
                    return false
                } else {
                    return true
                }
            },
            returnName(){
                if(this.tab == 'BRAND'){
                    return {
                        name: 'Brand Name',
                        photo: 'Brand Logo'
                    }
                } else if (this.tab == 'PARTS'){
                    return {
                        name: 'Parts Name',
                        photo: 'Parts Image',
                        short: 'Parts Short Description'
                    }
                } else if (this.tab == 'CAR MODELS'){
                    return {
                        name: 'Car Model Name',
                        photo: 'Car Model Image',
                        short: 'Short Description (Generations)'
                    }
                } else {
                    return {
                        name: ''
                    }
                }
            }

        },
        methods: {
            checkDialogOpen(){
                let tab = this.tab

                if(tab == 'BRAND' || tab == 'PARTS' || tab == 'CAR MODELS'){
                    this.showDialog = true
                } else {
                    this.showShopDialog = true
                }
            },
            uploadFilesViaHttp(file,size,folder){
                const fd = new FormData();
                let random = sri(9)
                fd.append('image', file, random+'.jpg')
                //IMPORTANT//
                // this sets the folder and resize guide//
                fd.append('folder',folder)
                fd.append('size',size)
                //IMPORTANT//
                return axios.post('https://us-central1-karmanila69-f7a2a.cloudfunctions.net/uploadFileResize', fd)
                .then(res => {
                    // console.log(res)
                    return res
                }).catch(res => {
                    // console.log('res',res.toString())    
                    let data = {
                            data: {
                                message: 'error'
                            }
                        }
                    return data
                })
            },  
            photoAddedGallery (p) {
                var photo = this.$refs.upldGallery.files
                this.uploadedPhoto = photo
            },
            photoAddedBrand (p) {
            // console.log(p[0].size,'size')
                var photo = this.$refs.upldBrand.files[0]
                // this.showImageError = false
                let vm = this
                if(this.$refs.upldBrand.files.length > 1){
                    let minus = vm.$refs.upldBrand.uploadSize - p[0].size
                    vm.$refs.upldBrand.uploadSize = minus
                    vm.$refs.upldBrand.files.splice(1,1)
                    vm.$refs.upldBrand.queuedFiles.splice(1,1)
                } 
                this.uploadedPhoto = photo
                // console.log('photo1', this.uploadedPhoto)
            },
            photoRemovedBrand () {
                this.uploadedPhoto = null
            },
            addNewRecord(){
                let location
                let data
                let size
                let collection
                // '300x300' for brand size
                // 'brands/' for brand location
                // 'brands' for brand collection

                if(this.tab == 'BRAND'){
                    location = 'brands/'
                    size = '300x300'
                    collection = 'brands'
                    data = {
                        name: this.newAdd.name,
                        type: this.newAdd.brandType,
                        photo: this.newAdd.photo,
                        dateAdded: new Date()
                    }
                } else if(this.tab == 'PARTS') {
                    location = 'parts/'
                    size = '500x500'
                    collection = 'parts'
                    data = {
                        name: this.newAdd.name,
                        photo: this.newAdd.photo,
                        shortDesc: this.newAdd.shortdesc,
                        dateAdded: new Date()
                    }
                } else if(this.tab == 'CAR MODELS') {
                    location = 'carModels/'
                    size = '500x500'
                    collection = 'carModels'
                    data = {
                        brand: this.newAdd.brand,
                        name: this.newAdd.name,
                        photo: this.newAdd.photo,
                        shortDesc: this.newAdd.shortdesc,
                        year: this.newAdd.year,
                        dateAdded: new Date()
                    }
                }

                this.$q.dialog({
                    title: 'Confirm Save',
                    message: 'Do you want to add this ' + this.tab +' ?',
                    color:'amber',
                    ok: 'Yes',
                    cancel: 'Cancel'
                }).onOk(() => {
                    this.loadingDialog = true
                    let vm = this
                    if(vm.$refs.upldBrand.files.length === 0){
                        vm.$q.notify({
                            message: `Please check file uploads.`,
                            type: 'negative',
                            color: 'negative',
                            textColor: 'white',
                            icon: 'negative'
                        })   
                        vm.loadingDialog = false
                    } else {
                        vm.fileUploadDatabaseAdd(collection,data,location,size)
                    }                    
                })
            },
            fileUploadDatabaseAdd(collection,data,location,size){
                let vm = this
                const fileUpload = vm.$refs.upldBrand.files[0]
                vm.uploadFilesViaHttp(fileUpload,size,location.toString())
                .then((img) => {
                    if(img.data.message === 'error'){
                        vm.$q.notify({
                            message: 'img=' + img.data.message,
                            type: 'negative',
                            color: 'negative',
                            textColor: 'white',
                            icon: 'negative'
                        }) 
                        vm.loadingDialog = false
                    } else {   
                        data.photo = img.data.link
                        vm.$firestoreApp.collection(collection).add(data)
                            .then(() => {
                                vm.newAddReset()
                                vm.showDialog = false
                                vm.loadingDialog = false
                                vm.$q.notify({
                                    message: this.tab +` Added Successfully`,
                                    type: 'positive',
                                    color: 'positive',
                                    textColor: 'white',
                                    icon: 'positive'
                                })
                                
                                console.log('ADDING SUCCESS')
                            })
                            .catch((err) => {
                                vm.newAddReset()
                                vm.showDialog = false
                                vm.$q.notify({
                                    message: err,
                                    type: 'negative',
                                    color: 'negative',
                                    textColor: 'white',
                                    icon: 'negative'
                                })   
                                console.log('ADDING ERROR')
                            })
                    }                             
                }) 
            },
            newAddReset(){
                this.newAdd.photo = ''
                this.newAdd.name = ''
                this.newAdd.shortdesc = ''
                this.newAdd.carModel = []
                this.newAdd.year = []
                this.newAdd.color = []
            }
        }

}
</script>