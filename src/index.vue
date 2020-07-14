<template>
    <my-list :list="mock" @load="handleLoad">
        <template v-slot="scope">{{ scope.item.label }}（{{ scope.item.value }}）</template>
    </my-list>
</template>

<script>
import MyList from "./components/List.vue";

export default {
    name: "App",
    components: {
        MyList
    },
    data() {
        return {
            total: 100
        };
    },
    computed: {
        mock() {
            let total = this.total;
            const result = [];
            
            while (total > 0) {
                result.push({
                    label: total,
                    value: total
                });
                
                total--;
            }
            
            return result.reverse();
        }
    },
    methods: {
        handleLoad() {
            this.total = this.total + 100;
        }
    }
};
</script>
