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
            // 数据格式（描述一个树节点，输入为root节点，嵌套描述整棵树）：
            // {
            //     "name": 树节点的id,
            //     "weight": 节点大小权重,
            //     "deep": 深度（root为0）,
            //     "class": 节点类别（非单个元素的叶节点类别为"group"),
            //     "_children": [] 节点的子节点list,
            //     可选：
            //     "category_cnt": {} 记录子树中每种类别的出现次数,
            // }
        };
    },
}
</script>
```
