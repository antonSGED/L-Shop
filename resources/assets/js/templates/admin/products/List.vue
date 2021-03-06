<template>
    <v-card>
        <v-card-title>
            {{ $t('content.admin.products.list.title') }}
            <v-btn flat color="primary" small icon :to="{name: 'admin.products.add'}"><v-icon>add</v-icon></v-btn>
            <v-spacer></v-spacer>
            <v-text-field
                    append-icon="search"
                    :label="$t('content.admin.products.list.search')"
                    single-line
                    hide-details
                    v-model="search"
            ></v-text-field>
        </v-card-title>
        <v-data-table
                :headers="headers"
                :items="items"
                :pagination.sync="pagination"
                :total-items="totalItems"
                :loading="loading"
                :no-data-text="$t('content.admin.products.list.table.empty')"
                :rows-per-page-items="[25, 50, 100]"
                :rows-per-page-text="$t('common.table.rows_per_page')"
                class="elevation-1"
        >
            <template slot="pageText" slot-scope="{ pageStart, pageStop }">
                {{ pageStart }} - {{ pageStop }} {{ $t('common.table.of') }} {{ totalItems }}
            </template>

            <template slot="items" slot-scope="props">
                <td class="text-xs-center">{{ props.item.id }}</td>
                <td class="text-xs-center"><img :src="props.item.item.image" height="30" :alt="props.item.item.name"></td>
                <td class="text-xs-center">{{ props.item.item.name }}</td>
                <td class="text-xs-center">
                    <v-enchanted class="cp" v-if="props.item.item.enchanted"></v-enchanted>
                    <v-hidden class="cp" v-if="props.item.hidden"></v-hidden>
                </td>
                <td class="text-xs-center">{{ props.item.item.type }}</td>
                <td class="text-xs-center">{{ props.item.price }} <span v-html="$store.state.shop.currency.html"></span></td>
                <td class="text-xs-center">{{ props.item.stack }}</td>
                <td class="text-xs-center">{{ props.item.server.name }}</td>
                <td class="text-xs-center">{{ props.item.category.name }}</td>
                <td class="justify-center layout px-0">
                    <v-btn icon class="mx-0" :to="{name: 'admin.products.edit', params: {product: props.item.id}}">
                        <v-icon color="secondary">edit</v-icon>
                    </v-btn>
                    <v-btn icon class="mx-0" @click="deleteProduct(props.item)">
                        <v-icon color="pink">delete</v-icon>
                    </v-btn>
                </td>
            </template>
        </v-data-table>
    </v-card>
</template>

<script>
    export default {
        data() {
            return {
                search: '',
                totalItems: 0,
                items: [],
                loading: false,
                pagination: {
                    page: this.$route.query.page ? this.$route.query.page : 1,
                    rowsPerPage: this.$route.query.per_page ? parseInt(this.$route.query.per_page) : 25,
                    sortBy: this.$route.query.order_by ? this.$route.query.order_by : 'product.id',
                    descending: this.$route.query.descending === 'true',
                },
                headers: [
                    {
                        text: $t('content.admin.products.list.table.headers.id'),
                        align: 'center',
                        sortable: true,
                        value: 'product.id'
                    },
                    {
                        text: $t('content.admin.products.list.table.headers.image'),
                        sortable: false,
                        align: 'center',
                    },
                    {
                        text: $t('content.admin.products.list.table.headers.item'),
                        align: 'center',
                        sortable: true,
                        value: 'item.name'
                    },
                    {
                        text: $t('common.note'),
                        align: 'center',
                        sortable: false,
                        value: 'note'
                    },
                    {
                        text: $t('content.admin.products.list.table.headers.type'),
                        align: 'center',
                        sortable: true,
                        value: 'item.type'
                    },
                    {
                        text: $t('content.admin.products.list.table.headers.price'),
                        align: 'center',
                        sortable: true,
                        value: 'product.price'
                    },
                    {
                        text: $t('content.admin.products.list.table.headers.stack'),
                        align: 'center',
                        sortable: true,
                        value: 'product.stack'
                    },
                    {
                        text: $t('common.server'),
                        align: 'center',
                        sortable: true,
                        value: 'server.name'
                    },
                    {
                        text: $t('common.category'),
                        align: 'center',
                        sortable: true,
                        value: 'category.name'
                    },
                    {
                        text: $t('common.actions'),
                        align: 'center',
                        sortable: false
                    }
                ]
            }
        },
        watch: {
            pagination: {
                handler () {
                    let query = {};
                    if (this.pagination.page) {
                        query.page = this.pagination.page;
                    }
                    if (this.pagination.rowsPerPage) {
                        query.per_page = this.pagination.rowsPerPage;
                    }
                    if (this.pagination.sortBy) {
                        query.order_by = this.pagination.sortBy;
                    }
                    if (this.pagination.descending !== null) {
                        query.descending = this.pagination.descending;
                    }


                    this.$router.replace({name: 'admin.products.list', query: query}, () => {
                        this.retrieveFromApi();
                    }, () => {
                        this.retrieveFromApi();
                    });
                },
                deep: true
            },
            search() {
                this.retrieveFromApi();
            }
        },
        methods: {
            retrieveFromApi() {
                this.loading = true;

                this.$axios.post('/spa/admin/products/list', {
                    page: this.$route.query.page,
                    per_page: this.$route.query.per_page,
                    order_by: this.$route.query.order_by,
                    descending: this.$route.query.descending,
                    search: this.search
                })
                    .then((response) => {
                        this.setTable(response.data)
                    });
            },
            deleteProduct(product) {
                if (confirm($t('content.admin.products.list.delete'))) {
                    this.$axios.post('/spa/admin/products', {
                        _method: 'DELETE',
                        product: product.id
                    })
                        .then(response => {
                            if (response.data.status === 'success') {
                                this.items.forEach((each, index) => {
                                    if (each.id === product.id) {
                                        this.items.splice(index, 1);
                                    }
                                });
                            }
                        })
                }
            },
            setTable(data) {
                this.totalItems = data.paginator.total;
                this.items = data.products;

                this.loading = false;
            }
        }
    }
</script>
