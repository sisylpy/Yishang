    <template>

        <li class="mytreeview">
            <a v-if="item.type === 0" href="javascript:;">
                <i v-if="item.icon != null" :class="item.icon" class="fa fa-link"></i>
                <span>{{item.name}}</span>
                <span class="pull-right-container">
                   <i class="fa fa-angle-left pull-right"></i>
            </span>
            </a>

            <ul v-if="item.type === 0" class="treeview-menu">
                <menu-item :item="item" v-for="item in item.list" v-bind:key="item.id"
                           @click.native="goTo(''+item.url+'',''+item.name+'')"></menu-item>
            </ul>
            <a v-if="item.type === 1">
                <i v-if="item.icon != null" :class="item.icon"></i>
                <i v-else class="fa fa-circle-o"></i> {{item.name}}</a>
        </li>


    </template>

    <script>
        export default {
            name: "menuItem",
            props: {
                item: {}
            },
            methods: {
                changeIcon: function () {

                },

                goTo: function (path,name) {
                    if(path.indexOf('?') > -1){
                        var arr = path.split('?');
                        var newPath = arr[0];
                        var param = arr[1];
                        var paramArr = param.split('=');
                        var paramValue = paramArr[1];
                        this.$router.replace({
                            name: newPath,
                            params: {
                                navTitle: name,
                                fatherId: paramValue
                            }
                        });
                    }else{
                        this.$router.replace({
                            name: path,
                            params: {
                                navTitle: name,
                            }
                        })
                    }

                    this.$store.dispatch('pageHeader/setNavTitle', name)

                }
            }

        }
    </script>

    <style>

    </style>
