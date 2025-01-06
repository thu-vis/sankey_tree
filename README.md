# Sankey Tree

1. 完成TODO：修改代码中定义的categories

2. 在vue中导入sankey_tree.vue中的template与模块，将数据传入组件，例如：
```vue
</template>
    <SankeyTree
        ref="sankey_tree"
        :item="your_data"
    />
</template>

<script>
import SankeyTree from './sankey_tree.vue'

export default {
    components: {
        SankeyTree,
    },
    data: function() {
        return {
            your_data: null,
        };
    },
}
</script>
```
