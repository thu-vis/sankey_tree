<template>
    <div class="svg-info V-centered" id="meta-svg-container-sankey">
        <div class="tree-box H-centered">
            <svg class="datainfo-svg" id="meta-svg-sankey" @contextmenu="disableRightClick">
                <g class="tree-g"/>
            </svg>
            <div id='tree_buttons' class='buttons'>
                <div id='tree_zoom_in' title='Zoom in' @click='buttonZoomIn' v-ripple class='button'>
                    <svg class='icon' width='30px' height='30px' transform='translate(0, 5)' viewBox='0 0 1024 1024'>
                        <path d="M754.2 151.5h-89.5v42.7h89.5c59.5 0 108 48.4 108 108v67.6h42.7v-67.6c0-83.1-67.6-150.7-150.7-150.7zM862.2 737.3c0 59.5-48.4 108-108 108h-89.5V888h89.5c83.1 0 150.7-67.6 150.7-150.7v-67.6h-42.7v67.6zM166.3 737.8v-67.6h-42.7v67.6c0 83.1 67.6 150.7 150.7 150.7h89.5v-42.7h-89.5c-59.5 0-108-48.4-108-108zM416.3 261.8h-42.8v126H247.6v42.7h125.9V556h42.8V430.5h125.4v-42.7H416.3z" fill="white" p-id="1775"></path><path d="M773.6 789.4l30.2-30.2-190.1-190.6c32.7-44.8 52-99.9 52-159.5 0-149.7-121.6-271.3-271.3-271.3-149.3 0-271.3 121.6-271.3 271.3 0 149.3 121.5 271.3 271.3 271.3 74.5 0 142.3-30.3 191.4-79.3l187.8 188.3zM394.4 637.7c-126 0-228.6-102.5-228.6-228.6s102.5-228.6 228.6-228.6S623 283.1 623 409.2 520.5 637.7 394.4 637.7z" fill="white" p-id="1776"></path>
                    </svg>
                </div>
                <div class='gap'></div>
                <div id='tree_zoom_out' title='Zoom out' @click='buttonZoomOut' v-ripple class='button'>
                    <svg class='icon' width='30px' height='30px' transform='translate(0, 5)' viewBox='0 0 1024 1024'>
                        <path d="M754.2 151.5h-89.5v42.7h89.5c59.5 0 108 48.4 108 108v67.6h42.7v-67.6c0-83.1-67.6-150.7-150.7-150.7zM862.2 737.3c0 59.5-48.4 108-108 108h-89.5V888h89.5c83.1 0 150.7-67.6 150.7-150.7v-67.6h-42.7v67.6zM166.3 737.8v-67.6h-42.7v67.6c0 83.1 67.6 150.7 150.7 150.7h89.5v-42.7h-89.5c-59.5 0-108-48.4-108-108zM247.6 387.8h294.2v42.7H247.6z" p-id="1938" fill="white"></path><path d="M773.6 789.4l30.2-30.2-190.1-190.6c32.7-44.8 52-99.9 52-159.5 0-149.7-121.6-271.3-271.3-271.3-149.3 0-271.3 121.6-271.3 271.3 0 149.3 121.6 271.3 271.3 271.3 74.5 0 142.3-30.3 191.4-79.3l187.8 188.3zM394.4 637.7c-126 0-228.6-102.5-228.6-228.6s102.5-228.6 228.6-228.6S623 283.1 623 409.2 520.5 637.7 394.4 637.7z" p-id="1939" fill="white"></path>
                    </svg>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import * as d3 from 'd3';
import { mapState, mapActions } from 'vuex';
window.d3 = d3;

export default {
    name: 'SankeyTree',
    components: {
    },
    props: ['item'],
    emits: ['click-node'],
    data: function() {
        return {
            svg: null,
            svg_g: null,
            svgsize: null,
            svg_width: 100,
            svg_height: 100,
            color_list: ['rgb(255, 23, 23)', 'rgb(255, 139, 23)', 'rgb(255, 238, 23)', 'rgb(180, 255, 23)', 'rgb(23, 255, 35)', 'rgb(23, 255, 255)', 'rgb(23, 64, 255)', 'rgb(168, 23, 255)', 'rgb(255, 83, 247)'],
            max_deep: 0,
            root: null,
            click_id: -1,
            click_ids: [],
            scale: 1,
            xshift: 0,
            yshift: 0,
            isShowHierarchy: false,
            is_zoom: 0,
            only_drag: false,
            remove_time: 100,
            update_time: 300,
            update_time: 200,
            timer: null,
            click_timer: null,
        };
    },
    methods: {
        disableRightClick(event) {
            event.preventDefault();
        },
        // 点击放大按钮
        buttonZoomIn: function() {
            let tmp = document.getElementById('meta-svg-sankey');
            if(this.is_zoom == 1) {
                tmp.classList.remove('magnifier-cursor');
                if(this.only_drag)
                    tmp.classList.add('drag-cursor');
                this.is_zoom = 0;
            }else {
                if(this.is_zoom == -1) {
                    tmp.classList.remove('shrink-cursor');
                    if(this.only_drag)
                        tmp.classList.add('drag-cursor');
                    this.is_zoom = 0;
                }
                if(this.only_drag)
                    tmp.classList.remove('drag-cursor');
                tmp.classList.add('magnifier-cursor');
                this.is_zoom = 1;
            }
        },
        // 点击缩小按钮
        buttonZoomOut: function() {
            let tmp = document.getElementById('meta-svg-sankey');
            if(this.is_zoom == -1) {
                tmp.classList.remove('shrink-cursor');
                if(this.only_drag)
                    tmp.classList.add('drag-cursor');
                this.is_zoom = 0;
            }else {
                if(this.is_zoom == 1) {
                    tmp.classList.remove('magnifier-cursor');
                    if(this.only_drag)
                        tmp.classList.add('drag-cursor');
                    this.is_zoom = 0;
                }
                if(this.only_drag)
                    tmp.classList.remove('drag-cursor');
                tmp.classList.add('shrink-cursor');
                this.is_zoom = -1;
            }
        },
        // 聚焦于id节点
        setClick: function(id) {
            if(id != -1) {
                if(this.timer != null) {
                    clearTimeout(this.timer);
                    this.timer = null;
                }
                this.click_id = id;
                this.nodeFullTmpExpand(this.h_data, id);
                this.resetNodeExpand(this.h_data);
                this.updateTree();
            } else {
                if(this.timer != null) {
                    clearTimeout(this.timer);
                    this.timer = null;
                }
                this.timer = setTimeout(() => {
                    this.click_id = id;
                    this.nodeFullTmpExpand(this.h_data, id);
                    this.resetNodeExpand(this.h_data);
                    this.updateTree();
                }, 10);
            }
        },
        // 设置是否展示hierarchy
        setShowHierarchy: function(show) {
            this.isShowHierarchy = show;
            this.updateTree();
        },
        // 通过id找到node
        getClickIds: function(id) {
            function dfs_find_ids(node, id) {
                if(node.name == id) {
                    return node.child_names;
                }
                for(let child of node._children) {
                    let ret = dfs_find_ids(child, id);
                    if(ret != null)return ret;
                }
                return null;
            }
            return dfs_find_ids(this.h_data, id);
        },
        // 更新sankey tree svg
        updateTree: function(is_zoom=false) {
            let that = this;
            if(that.svg == null)return;

            if (false) {
            // if (that.isShowHierarchy) {
                this.legend.attr("visibility", "hidden");
                this.legend_hierarchy.attr("visibility", null);
            } else {
                this.legend.attr("visibility", null);
                this.legend_hierarchy.attr("visibility", "hidden");
            }

            that.updateTreeLayout();

            let node_basesize = min(this.level_dist/5, this.svg_height/25);

            this.width_ratio = min(this.width_ratio, node_basesize*2/that.h_data.min_weight)
            
            this.svg_g.attr("transform", "translate("+this.xshift+","+this.yshift+")");

            that.click_ids = [];

            let tmp_ids = that.getClickIds(that.click_id);
            if(tmp_ids != null)
                that.click_ids = tmp_ids;

            let lineGenerator = d3.line().curve(d3.curveBasis);

            function get_source_x(node, i, name) {
                if(node.data.cnt==0) return [node.x*that.scale, 0];
                let cls = that.categories[i];
                let cnt = 0;
                let cnt2 = 0;
                for(let j=0;j<i;j++){
                    let cls2 = that.categories[j];
                    cnt += node.data.category_cnt[cls2["name2"]]/node.data.cnt*node.data.width_weight;
                }
                cnt2 = cnt;
                // console.log("cnt1", cnt, cnt2, node.data);
                for(let child of node.data.children) {
                    if(child.name == name) {
                        cnt2 = cnt + child.category_cnt[cls["name2"]]/node.data.cnt*node.data.width_weight;
                        break;
                    }
                    cnt += child.category_cnt[cls["name2"]]/node.data.cnt*node.data.width_weight;
                    cnt2 = cnt;
                }
                // console.log("cnt2", cnt, cnt2);
                let tmp_width = node.data.width_weight*that.width_ratio*Math.sqrt(that.scale);
                let tmp_x = node.x*that.scale+((cnt+cnt2)/2)*that.width_ratio*Math.sqrt(that.scale)-tmp_width/2;
                return [tmp_x, (cnt2-cnt)*that.width_ratio*Math.sqrt(that.scale)];
            }
            
            function get_target_x(node, i) {
                if(node.data.cnt==0) return node.x*that.scale;
                let cls = that.categories[i];
                let cnt = 0;
                for(let j=0;j<i;j++){
                    let cls2 = that.categories[j];
                    cnt += node.data.category_cnt[cls2["name2"]]/node.data.cnt*node.data.width_weight;
                }
                let cnt2 = cnt + node.data.category_cnt[cls["name2"]]/node.data.cnt*node.data.width_weight;
                let tmp_width = node.data.width_weight*that.width_ratio*Math.sqrt(that.scale);
                let tmp_x = node.x*that.scale+((cnt+cnt2)/2)*that.width_ratio*Math.sqrt(that.scale)-tmp_width/2;
                return tmp_x;
            }

            function get_key_width(node, i) {
                if(node.data.cnt==0) return 0;
                let cls = that.categories[i];
                let tmp_width = node.data.category_cnt[cls["name2"]]/node.data.cnt*node.data.width_weight*that.width_ratio*Math.sqrt(that.scale);
                return tmp_width;
            }

            for(let i=0;i<that.categories.length;i++) {
                let cls = that.categories[i];
                let links = that.svg_g.selectAll(`.${cls["name2"]}-link`)
                    .data(that.link_data, d => { return String(d.source.data.name)+"&"+String(d.target.data.name); });
                links.attr("stroke", d => {
                        let f_data = d.source, c_data = d.target;
                        if(f_data.y > c_data.y) { c_data = d.source; f_data = d.target; }
                        if(get_key_width(c_data, i)>0)
                            return "gray";
                        return "none";
                    })
                    .attr("fill",  d => {
                        let f_data = d.source, c_data = d.target;
                        if(f_data.y > c_data.y) { c_data = d.source; f_data = d.target; }
                        if(get_key_width(c_data, i)>0)
                            return cls["color"];
                        return "none";
                    })
                    .attr("visibility", function(d) {
                        if(d.source.data.class == "empty")return "hidden";
                        if(d.target.data.class == "empty")return "hidden";
                        return null;
                    })
                    .transition()
                    .duration(is_zoom?0:that.update_time)
                    .delay(is_zoom?0:that.remove_time)
                    .attr("d", d => {
                        let f_data = d.source, c_data = d.target;
                        if(f_data.y > c_data.y) { c_data = d.source; f_data = d.target; }
                        let [source_x, tmp_width2] = get_source_x(f_data, i, c_data.data.name);
                        let target_x = get_target_x(c_data, i);
                        let tmp_width = get_key_width(c_data, i);
                        let f_bias = node_basesize;
                        let c_bias = node_basesize;
                        // console.log("bias", source_x, tmp_width2, target_x, tmp_width, f_bias, c_bias)
                        const source1 = [f_data.y*that.scale+f_bias*Math.sqrt(that.scale), source_x-tmp_width2/2];
                        const target1 = [c_data.y*that.scale-c_bias*Math.sqrt(that.scale), target_x-tmp_width/2];
                        const source2 = [f_data.y*that.scale+f_bias*Math.sqrt(that.scale), source_x+tmp_width2/2];
                        const target2 = [c_data.y*that.scale-c_bias*Math.sqrt(that.scale), target_x+tmp_width/2];

                        let line1 = lineGenerator([source1, [(source1[0]*2+target1[0])/3, source1[1]], [(source1[0]+target1[0]*2)/3, target1[1]], target1]);
                        let line2 = lineGenerator([source2, [(source2[0]*2+target2[0])/3, source2[1]], [(source2[0]+target2[0]*2)/3, target2[1]], target2].reverse());
                        return line1 + "L" + line2.slice(1) + "Z";
                    })
                    // .attr("stroke-width", d => {
                    //     let f_data = d.source, c_data = d.target;
                    //     if(f_data.y > c_data.y) { c_data = d.source; f_data = d.target; }
                    //     return get_key_width(c_data, i);
                    // })
                    .attr('opacity', function(d) {
                        if(that.click_ids.length>0){
                            if((that.click_ids.includes(d.source.data.name))&&(that.click_ids.includes(d.target.data.name)))
                                return 0.5;
                            else return 0.25;
                        }
                        return 0.5;
                    });

            }

            const node = that.svg_g.selectAll(".node")
                .data(that.node_data, d=>d.data.name);
            
            node.transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("transform", d => `translate(${d.y*that.scale},${d.x*that.scale})`)
                .attr('opacity', function(d) {
                    if(that.click_ids.length>0){
                        if(that.click_ids.includes(d.data.name))
                            return 1;
                        else return 0.5;
                    }
                    return 1;
                });

            node.filter((d) => that.click_id == d.data.name).raise();

            node.select("circle")
                .attr("fill",  (d) => {
                    if(d.data.class == "group")
                        return 'white';
                    else if(d.data.class in that.categories_dict)
                        return that.categories_dict[d.data.class]["color"];
                    else return "transparent";
                })
                .attr("stroke", function(d) {
                    if(that.click_id == d.data.name)
                        return "black";
                    return "gray";
                })
                .attr("stroke-width", function(d) {
                    if(that.click_id == d.data.name)
                        return that.width_ratio/4*Math.sqrt(that.scale);
                    return that.width_ratio/12*Math.sqrt(that.scale);
                })
                .attr("stroke-dasharray", function(d) {
                    return null;
                })
                .attr("visibility", function(d) {
                    return "hidden";
                    if(d.data.class == "group")
                        return "visible";
                    return "hidden";
                })
                .transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("opacity",  function(d) {
                    if((d.data.class == "group")&&(d.data.expand||d.data.tmp_expand))
                        return 1;
                    return 0;
                })
                .attr("r", function(d) {
                    if(that.click_id == d.data.name)
                        return that.width_ratio*3/4*Math.sqrt(that.scale);
                    return that.width_ratio/2*Math.sqrt(that.scale);
                });

            node.select("rect")
                .attr("fill",  (d) => {
                    if(d.data.class == "group")
                        return 'white';
                    else if(d.data.class in that.categories_dict)
                        return that.categories_dict[d.data.class]["color"];
                    else return "transparent";
                })
                .attr("stroke", function(d) {
                    if(that.click_id == d.data.name)
                        return "black";
                    return "gray";
                })
                .attr("stroke-dasharray", function(d) {
                    return null;
                })
                .attr("visibility", function(d) {
                    if(d.data.class != "group" && d.data.class != "empty")
                        return "visible";
                    return "hidden";
                })
                .transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("opacity",  function(d) {
                    if(d.data.class != "group")
                        return 1;
                    return 0;
                })
                .attr("width", function(d) {
                    if(that.click_id == d.data.name)
                        return node_basesize*1.2*Math.sqrt(that.scale)*2;
                    return node_basesize*Math.sqrt(that.scale)*2;
                })
                .attr("height", function(d) {
                    if(that.click_id == d.data.name)
                        return d.data.width_weight*that.width_ratio/2*1.2*Math.sqrt(that.scale)*2;
                    return d.data.width_weight*that.width_ratio/2*Math.sqrt(that.scale)*2;
                })
                .attr("x", function(d) {
                    if(that.click_id == d.data.name)
                        return -node_basesize*1.2*Math.sqrt(that.scale);
                    return -node_basesize*Math.sqrt(that.scale);
                })
                .attr("y", function(d) {
                    if(that.click_id == d.data.name)
                        return -d.data.width_weight*that.width_ratio/2*1.2*Math.sqrt(that.scale);
                    return -d.data.width_weight*that.width_ratio/2*Math.sqrt(that.scale);
                });

            node.select(".polygon")
                .attr("fill",  (d) => {
                    if(d.data.class == "group")
                        return 'white';
                    else if(d.data.class in that.categories_dict)
                        return that.categories_dict[d.data.class]["color"];
                    else return "transparent";
                })
                .attr("stroke", function(d) {
                    if(that.click_id == d.data.name)
                        return "black";
                    return "gray";
                })
                // .attr("stroke-width", function(d) {
                //     if(that.click_id == d.data.name)
                //         return that.width_ratio/4*Math.sqrt(that.scale);
                //     return that.width_ratio/12*Math.sqrt(that.scale);
                // })
                .attr("stroke-dasharray", function(d) {
                    return null;
                })
                .transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("points", (d) => {
                    let width = that.width_ratio*d.data.width_weight*Math.sqrt(that.scale);
                    let height = 2*node_basesize*Math.sqrt(that.scale);
                    let bias_y = -node_basesize*Math.sqrt(that.scale);
                    return bias_y+","+width/2+" "+(height+bias_y)+","+width/2+" "+(height+bias_y)+","+(-width/2)+" "+bias_y+","+(-width/2);
                });
            
            for (let i=0; i<that.categories.length; i++) {
                let cls = that.categories[i];
                node.select(`.${cls["name2"]}-polygon`)
                    .transition()
                    .duration(is_zoom?0:that.update_time)
                    .delay(is_zoom?0:that.remove_time)
                    .attr("points", (d) => {
                        let width = that.width_ratio*d.data.width_weight*Math.sqrt(that.scale);
                        let c_width = 0;
                        if(d.data.cnt>0)
                            c_width = that.width_ratio*d.data.width_weight*d.data.category_cnt[cls["name2"]]/d.data.cnt*Math.sqrt(that.scale);
                        let begin_x = 0;
                        for(let j=0; j<i; j++) {
                            let cls2 = that.categories[j];
                            if(d.data.cnt>0)
                                begin_x += that.width_ratio*d.data.width_weight*d.data.category_cnt[cls2["name2"]]/d.data.cnt*Math.sqrt(that.scale);
                        }
                        let height = 2*node_basesize*Math.sqrt(that.scale);
                        let bias_y = -node_basesize*Math.sqrt(that.scale);
                        return bias_y+","+(-width/2+begin_x)+" "+(height+bias_y)+","+(-width/2+begin_x)+" "+(height+bias_y)+","+(-width/2+begin_x+c_width)+" "+bias_y+","+(-width/2+begin_x+c_width);
                    });
                }
            
            node.select(".key-text-g")
                .attr("transform", function(d) {
                    let bias_y = 0*node_basesize*Math.sqrt(that.scale);
                    let bias_x = node_basesize;
                    return `translate(${bias_x}, ${bias_y})`;
                });
            
            node.select(".poly_text")
                .transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("dy", ".35em")
                .attr("font-size", function(d) {
                    let height = min(2*node_basesize, that.width_ratio*d.data.width_weight)*Math.sqrt(that.scale);
                    return `${height}px`;
                });

            node.select(".poly_g")
                .attr("visibility", function(d) {
                    if(d.data.class == "group")
                        return "visible";
                    return "hidden";
                })
                .transition()
                .duration(is_zoom?0:that.update_time)
                .delay(is_zoom?0:that.remove_time)
                .attr("opacity",  function(d) {
                    if((d.data.class == "group")&&(!d.data.expand)&&(!d.data.tmp_expand))
                        return 1;
                    if(d.data.class == "group")
                        return 1;
                    return 0;
                });
        },
        //右侧插入空节点（layout算法占位）
        InsertEmptyRight(node, last_dict) {
            let that = this;
            let nodeCopy = { ...node };
            if (nodeCopy.children && nodeCopy.children.length>0) {
                nodeCopy.children = [...nodeCopy.children];
                for (let i = nodeCopy.children.length - 1; i >= 0; i--) {
                    nodeCopy.children[i] = that.InsertEmptyRight(nodeCopy.children[i], last_dict);
                }
                if (!(nodeCopy.deep in last_dict)) {
                    last_dict[nodeCopy.deep] = nodeCopy;
                    let category_cnt = {};
                    for(let key in that.categories_dict) category_cnt[key] = 0;
                    let emptyNode = {"name": Math.floor(Math.random()*2147483647), "deep": 0, "class": "empty", "child_names": [], "child_names2": [], "expand": false, "_children": [], "tmp_expand": false, "category_cnt": category_cnt, "cnt": 0, "layers": 0, "width_weight": 0};
                    nodeCopy.children = [...nodeCopy.children, emptyNode];
                }
            }
            return nodeCopy;
        },
        //左侧插入空节点（layout算法占位）
        InsertEmptyLeft(node, first_dict) {
            let that = this;
            let nodeCopy = { ...node };
            if (nodeCopy.children && nodeCopy.children.length>0) {
                nodeCopy.children = [...nodeCopy.children];
                for (let i = 0; i < nodeCopy.children.length; i++) {
                    nodeCopy.children[i] = that.InsertEmptyLeft(nodeCopy.children[i], first_dict);
                }
                if (!(nodeCopy.deep in first_dict)) {
                    first_dict[nodeCopy.deep] = nodeCopy;
                    let category_cnt = {};
                    for(let key in that.categories_dict) category_cnt[key] = 0;
                    let emptyNode = {"name": Math.floor(Math.random()*2147483647), "deep": 0, "class": "empty", "child_names": [], "child_names2": [], "expand": false, "_children": [], "tmp_expand": false, "category_cnt": category_cnt, "cnt": 0, "layers": 0, "width_weight": 0};
                    nodeCopy.children = [emptyNode, ...nodeCopy.children];
                }
            }
            return nodeCopy;
        },
        // 更新sankey tree svg layout
        updateTreeLayout() {
            let that = this;
            let h_data = that.h_data;
            if(h_data != null) {
                let tmp_h_data = that.InsertEmptyRight(that.InsertEmptyLeft(h_data, {}), {});
                // console.log("h_data", h_data, tmp_h_data);
                const root = d3.hierarchy(tmp_h_data);
                const treeLayout = d3.tree().size([0.825*that.svg_height, 0.625*that.svg_width])
                .separation(function (a, b) {
                    let length = (a.data.width_weight+b.data.width_weight)/2;
                    if(a.data.class != "empty") length += 0.5*h_data.min_weight;
                    if(b.data.class != "empty") length += 0.5*h_data.min_weight;
                    return length;
                });

                treeLayout(root);
                
                that.root = root;

                let lineGenerator = d3.line().curve(d3.curveBasis);

                that.link_data = root.links();
                
                for (let cls of that.categories) {
                    let links = that.svg_g.selectAll(`.${cls["name2"]}-link`)
                        .data(that.link_data, d => { return String(d.source.data.name)+"&"+String(d.target.data.name); });
                    // console.log("links", links, root.links());
                    links.enter()
                        .append("path")
                        .attr("class", `${cls["name2"]}-link`)
                        .attr("opacity", 0)
                        .attr("d", d => {
                            const source = [d.source.y*that.scale, d.source.x*that.scale];
                            const target = [d.target.y*that.scale, d.target.x*that.scale];
                            return lineGenerator([source, [source[0], (source[1]*2+target[1])/3], [target[0], (source[1]+target[1]*2)/3], target]);
                        })
                        .transition()
                        .duration(that.create_time)
                        .delay(that.remove_time+that.update_time)
                        .attr("opacity", 1);
                    links.exit()
                        .transition()
                        .duration(that.remove_time)
                        .attr("opacity", 0)
                        .remove();
                }

                // console.log('root', root.descendants());
                that.node_data = root.descendants();
                // console.log("root", that.node_data);
                const node = that.svg_g.selectAll(".node")
                    .data(that.node_data, d=>d.data.name);
                
                that.level_dist = 0.85*that.svg_height;
                for(let node of that.node_data) {
                    if (node.parent) {
                        that.level_dist = Math.abs(node.y - node.parent.y);
                        break;
                    }
                };
                that.width_ratio = 0.7*that.svg_height/root.data.width_weight;
                for(let i=0;i<that.node_data.length-1;i++)
                for(let j=i+1;j<that.node_data.length;j++) {
                    let node_i = that.node_data[i];
                    let node_j = that.node_data[j];
                    if(node_i.depth != node_j.depth)continue;
                    let tot_length = (node_i.data.width_weight+node_j.data.width_weight)/2;
                    if(node_i.data.class != "empty") tot_length += 0.5*h_data.min_weight;
                    if(node_j.data.class != "empty") tot_length += 0.5*h_data.min_weight;
                    if(tot_length<0.25)continue;
                    // that.width_ratio = Math.min(that.width_ratio, 0.8*Math.abs(node_i.x - node_j.x)/tot_length);
                    that.width_ratio = Math.min(that.width_ratio, 0.8*Math.abs(node_i.x - node_j.x)/tot_length);
                    // console.log("width_ratio", that.width_ratio, node_i.x, node_j.x, tot_length, node_i, node_j);
                }

                const new_node = node.enter()
                    .append("g")
                    .attr("class", "node")
                    .attr("opacity", 0)
                    .attr("transform", d => `translate(${d.x*that.scale},${d.y*that.scale})`)
                    .on('click', function(e, d) {
                        if(that.click_timer != null) {
                            clearTimeout(that.click_timer);
                            that.click_timer = null;
                        }
                        that.click_timer = setTimeout(function() {
                            // console.log("click");
                            if(that.click_id != d.data.name) {
                                // console.log(d);
                                that.click_id = d.data.name;
                                that.$emit('click-node', d.data.name);
                                // that.nodeExpand(that.h_data, d.data.name);
                                that.nodeFullExpand(that.h_data, d.data.name);
                                that.nodeTrim(that.h_data, d.data.name);
                                
                                that.resetNodeExpand(that.h_data);
                                that.updateTree();
                            }else {
                                that.click_id = -1;
                                that.$emit('click-node', -1);
                                that.nodeClose(that.h_data, d.data.name);
                                that.resetNodeExpand(that.h_data);
                                that.updateTree();
                            }
                        }, 200);
                    })
                    .on('dblclick', function(e, d) {
                        // console.log("dblclick");
                        if(that.click_timer != null) {
                            clearTimeout(that.click_timer);
                            that.click_timer = null;
                        }
                        that.click_id = d.data.name;
                        that.$emit('click-node', d.data.name);
                        that.nodeFullExpand(that.h_data, d.data.name);
                        that.resetNodeExpand(that.h_data);
                        that.updateTree();
                    })
                    .style("cursor", "pointer");
                new_node.transition()
                    .duration(that.create_time)
                    .delay(that.remove_time+that.update_time)
                    .attr("opacity", 1);
                new_node.append("circle")
                    .attr("opacity", 0);
                new_node.append("rect")
                    .attr("opacity", 0);
                
                let key_text_g = new_node.append("g")
                    .attr("class", "key-text-g")
                    .attr("opacity", 0)
                    .attr("visibility", "hidden");

                key_text_g.selectAll(".key-text")
                    .data(d => Object.entries(d.data.category_cnt).filter(([key, value]) => value !== 0))
                    .enter()
                    .append("text")
                    .attr("class", "key-text")
                    .text(([key, value]) => `${value}`)
                    // .attr("x", 0)
                    .attr("x", (d, i) => 5 + i * 40)
                    .attr("y", 0)
                    // .attr("y", (d, i) => i * 40)
                    .attr("dy", ".35em")
                    .attr("fill", function([key, value]) {
                        if(key == "group")
                            return 'transparent';
                        else if(key in that.categories_dict)
                            return that.categories_dict[key]["color"];
                        else return "transparent";
                    })  
                    .attr("font-size", "30px")
                    .attr("stroke-width", 1)
                    .attr("stroke", 'black');

                const polyHover = function(e, d) {
                    const el = that.svg.selectAll(".key-text-g").filter((d2) => d2.data.name == d.data.name);
                    // console.log(el, d.data.name);
                    const isEntering = e.type === 'mouseenter';
                    d3.select(el.node()).raise();
                    d3.select(el.node().parentNode).raise();
                    if(isEntering) {
                        d3.select(el.node())
                            .attr("visibility", null)
                            .transition()
                            .duration(that.update_time)
                            .attr("opacity", 1);
                    }else {
                        d3.select(el.node())
                            .transition()
                            .duration(that.update_time)
                            .attr("opacity", 0)
                            .on('end', function(d) {
                                d3.select(this).attr("visibility", "hidden");
                            });
                    }
                };

                let poly_g = new_node.append("g")
                    .attr("class", "poly_g")
                    .attr("opacity", 0)
                    .on('mouseenter mouseleave', polyHover);
                poly_g.append("polygon")
                    .attr("class", "polygon");
                for (let cls of that.categories) {
                    poly_g.append("polygon")
                        .attr("class", d => `${cls["name2"]}-polygon`)
                        .attr("fill", function(d) {
                            return cls["color"];
                        })
                        .on("mouseover", function(d) {
                            d3.select(this).style("stroke", "black");
                        })
                        .on("mouseout", function(d) {
                            d3.select(this).style("stroke", null);
                        });
                };
                poly_g.append('text')
                    .attr("class", "poly_text")
                    .attr("text-anchor", "middle")
                    .text(d => String(d.data.cnt));
                
                node.exit()
                    .transition()
                    .duration(that.remove_time)
                    .attr("opacity", 0)
                    .remove();

                that.svg.on("click", function(event) {
                    const rect = that.svg.node().getBoundingClientRect();
                    const x = event.clientX - rect.left;
                    const y = event.clientY - rect.top;
                    if(that.is_zoom!=0) {
                        that.ImageZoom(x, y, 2*that.is_zoom);
                        return;
                    }
                    if (event.target.tagName === "svg") {
                        that.click_id = -1;
                        that.$emit('click-node', -1);
                        that.updateTree();
                    }
                });
            }
        },
        // 展开节点
        nodeExpand(node, name) {
            let find = false;
            if(node.name == name)
                find = true
            for(let child of node._children) {
                if(this.nodeExpand(child, name))
                    find = true;
            }
            if(find) {
                node.expand = true;
            }
            return find;
        },
        // 展开节点下的所有子树
        nodeFullExpand(node, name, father_find=false) {
            if(node.name == name)
                father_find = true
            let find = father_find;
            for(let child of node._children) {
                if(this.nodeFullExpand(child, name, father_find))
                    find = true;
            }
            if(find) {
                node.expand = true;
            }
            return find;
        },
        // 临时地展开至某个节点
        nodeFullTmpExpand(node, name, father_find=false) {
            if(node.name == name)
                father_find = true
            let find = father_find;
            for(let child of node._children) {
                if(this.nodeFullTmpExpand(child, name, father_find))
                    find = true;
            }
            if(find) {
                node.tmp_expand = true;
            }else {
                node.tmp_expand = false;
            }
            return find;
        },
        // 关闭节点
        nodeClose(node, name, find=false) {
            if(node.name == name)
                find = true;
            for(let child of node._children) {
                this.nodeClose(child, name, find);
            }
            if(find) {
                node.expand = false;
            }
        },
        // 设置节点展开情况
        resetNodeExpand(node) {
            // console.log(node);

            // if(node.class == "group") node.width_weight = 2;
            // else node.width_weight = 1;
            node.width_weight_o = node.weight;
            node.width_weight = Math.sqrt(node.width_weight_o);
            // node.width_weight = node.width_weight_o;

            if(node.expand||node.tmp_expand) {
                node.children = node._children;
            }else {
                node.children = null;
            }
            for(let child of node._children) {
                this.resetNodeExpand(child);
            }

            node.min_weight = node.width_weight;
            if(node._children != null && node._children.length>0) {
                let tmp_weight_o = 0;
                for(let child of node._children) {
                    tmp_weight_o += child.width_weight_o;
                }
                // node.width_weight_o = max(node.width_weight_o, tmp_weight_o);
                node.width_weight_o = tmp_weight_o;

                node.width_weight = Math.sqrt(node.width_weight_o);
                // node.width_weight = node.width_weight_o;
                node.min_weight = node.width_weight;
            }
            if(node.children != null && node.children.length>0) {
                for(let child of node.children) {
                    node.min_weight = min(node.min_weight, child.min_weight);
                }
            }
        },
        // 计算距离
        nodeDistUp(node, name) {
            node.dist = 10000;
            if(node.name == name) {
                node.dist = 0;
            }
            for(let child of node._children) {
                let child_dist = this.nodeDistUp(child, name)
                node.dist = min(node.dist, child_dist+1);
            }
            return node.dist;
        },
        // 计算距离
        nodeDistDown(node, father_dist) {
            node.dist = min(node.dist, father_dist+1);
            for(let child of node._children) {
                this.nodeDistDown(child, node.dist)
            }
        },
        nodeDfs(node, name) {
            let find = false;
            if(node.name == name)
                find = true;
            let node_list = [];
            let expanded_node_list = [];
            if(node.expand) {
                for(let child of node._children) {
                    let [child_node_list, child_expanded_node_list, child_find] = this.nodeDfs(child, name);
                    node_list = node_list.concat(child_node_list);
                    expanded_node_list = expanded_node_list.concat(child_expanded_node_list);
                    if(child_find)
                        find = true;
                }
                if(!find)
                    expanded_node_list.push(node)
            }
            node_list.push(node)
            return [node_list, expanded_node_list, find];
        },
        // 将展开的部分剪枝收起
        nodeTrim(h_data, name, max_cnt=10) {
            this.nodeDistUp(h_data, name);
            this.nodeDistDown(h_data, 10000);
            while(true) {
                let [node_list, expanded_node_list, find] = this.nodeDfs(h_data, name);
                // console.log("node number", node_list, expanded_node_list);
                if((node_list.length>max_cnt)&&(expanded_node_list.length>0)) {
                    let now_node = null;
                    for(let node of expanded_node_list) {
                        if((now_node == null)||(node.dist>now_node.dist)||((node.dist==now_node.dist)&&(node.child_names.length<now_node.child_names.length)))
                            now_node = node;
                    }
                    this.nodeClose(now_node, now_node.name);
                }else break;
            }
        },
        // 放大/缩小
        ImageZoom(x, y, zoom) {
            let that = this;
            const scaleBefore = that.scale;
            const zoomFactor = 0.2;
            that.scale *= 1+zoom*zoomFactor;
            that.scale = Math.max(1, that.scale);

            that.xshift = x + (that.xshift - x)*that.scale/scaleBefore;
            that.yshift = y + (that.yshift - y)*that.scale/scaleBefore; 

            that.updateTree(true);
        }

    },
    computed: {
    },
    watch: {
        // TODO: item格式（描述一个树节点，输入为root节点，嵌套描述整棵树）：
        // {
        //     "name": 树节点的id,
        //     "weight": 节点大小权重,
        //     "deep": 深度（root为0）,
        //     "class": 节点类别（非单个元素的叶节点类别为"group"),
        //     "_children": [] 节点的子节点list,
        //     可选：
        //     "category_cnt": {} 记录子树中每种类别的出现次数,
        // }
        item: function(item) {
            let that = this;
            if(that.svg == null)return;

            that.scale = 1;
            that.xshift = 0.275*that.svg_width;
            that.yshift = 0.05*that.svg_height;
            this.svg_g.attr("transform", "translate("+this.xshift+","+this.yshift+")");

            that.svg.on("wheel", (event) => {
                const rect = that.svg.node().getBoundingClientRect();
                const x = event.clientX - rect.left;
                const y = event.clientY - rect.top;
                that.ImageZoom(x, y, event.deltaY<0 ? 1 : -1);
            });

            let drag_offsetX = 0;
            let drag_offsetY = 0;

            const ImageMoveStartEnd = function(e, d) {
                if(e.type=='start') {
                    drag_offsetX = e.x - that.xshift;
                    drag_offsetY = e.y - that.yshift;
                }
            };

            const ImageMoving = function(e, d) {
                that.xshift = e.x - drag_offsetX;
                that.yshift = e.y - drag_offsetY;
                that.updateTree();
            };

            that.svg.call(d3.drag()
                .container(that.svg.node())
                .on('start end', ImageMoveStartEnd)
                .on('drag', ImageMoving),
            );

            // console.log("item", item);
            
            that.max_deep = 0;

            let h_data = null;
            function dfs(hierarchy) {  // 更新统计信息
                // hierarchy.weight = hierarchy.area;
                // delete hierarchy.category_cnt;

                if((hierarchy._children == null)||(hierarchy._children.length == 0)) {
                    let category_cnt = {};
                    for(let key in that.categories_dict) category_cnt[key] = 0;
                    if("category_cnt" in hierarchy) category_cnt = hierarchy["category_cnt"];
                    else category_cnt[hierarchy.class] = 1;

                    let tmp_cnt = Object.values(category_cnt).reduce((acc, val) => acc + val, 0);
                    return {"name": hierarchy.name, "weight": hierarchy.weight, "deep": hierarchy.deep, "class": hierarchy.class, "child_names": [hierarchy.name], "child_names2": [hierarchy.name], "expand": false, "_children": [], "tmp_expand": false, "category_cnt": category_cnt, "cnt": tmp_cnt, "layers": 0};
                }
                let category_cnt = {};
                for(let key in that.categories_dict) category_cnt[key] = 0;
                let tmp = {
                    name: hierarchy.name,
                    weight: hierarchy.weight,
                    deep: hierarchy.deep,
                    class: hierarchy.class,
                    _children: [],
                    children: null,
                    expand: false,
                    tmp_expand: false,
                    child_names: [hierarchy.name],
                    child_names2: [],
                    category_cnt: category_cnt,
                    layers: 0,
                }
                that.max_deep = Math.max(that.max_deep, hierarchy.deep);
                for(let child of hierarchy._children) {
                    let tmp_child = dfs(child);
                    tmp._children.push(tmp_child);
                    tmp.child_names = tmp.child_names.concat(tmp_child.child_names);
                    tmp.child_names2 = tmp.child_names2.concat(tmp_child.child_names2);
                    for(let key in tmp_child.category_cnt) {
                        tmp.category_cnt[key] += tmp_child.category_cnt[key];
                    }
                    tmp.layers = max(tmp.layers, tmp_child.layers+1);
                }
                if("category_cnt" in hierarchy) tmp.category_cnt = hierarchy["category_cnt"];

                tmp.cnt = Object.values(tmp.category_cnt).reduce((acc, val) => acc + val, 0);
                return tmp;
            }

            h_data = dfs(item);

            // that.nodeExpand(h_data, h_data.name);
            that.nodeFullExpand(h_data, h_data.name);
            that.nodeTrim(h_data, h_data.name);

            that.resetNodeExpand(h_data);
            console.log("hierarchy", h_data);

            
            for (let cls of that.categories) {
                that.svg_g.selectAll(`.${cls["name2"]}-link`).remove();
            }
            that.svg_g.selectAll(".node").remove();

            that.h_data = h_data;
            that.updateTree();

        }
    },
    mounted() {
        this.svg = d3.select("#meta-svg-sankey");
        this.svgsize = this.svg.node().getBoundingClientRect();
        this.svg_width = this.svgsize.width;
        this.svg_height = this.svgsize.height;

        this.scale = 1;
        this.xshift = 0.075*this.svg_width;
        this.yshift = 0.05*this.svg_height;
        this.svg_g = this.svg.select(".tree-g")
            .attr("transform", "translate("+this.xshift+","+this.yshift+")");

        let legend_size = 30;
        legend_size = min(30, this.svg_height/10);

        // TODO: change to your categories
        // name为显示的类名，name2为存储的类名，color为颜色
        const categories = [
            { name: "Data Element", name2: "data_element", color: 'rgb(44, 160, 44)' },
            { name: "Non-Data Element", name2: "non-data_element", color: 'rgb(214, 39, 40)' },
            { name: "Visual Element", name2: "visual_element", color: 'rgb(255, 127, 14)' },
            { name: "Text", name2: "text", color: 'rgb(31, 119, 180)' }
        ];

        this.categories = categories;
        this.categories_dict = {};
        for(let item of this.categories) {
            this.categories_dict[item["name2"]] = item;
        }

        const legend = this.svg.selectAll(".legend")
            .data(categories)
            .enter().append("g")
            .attr("class", "legend")
            .attr("transform", (d, i) => `translate(0, ${i * legend_size})`); // 设定每个图例的垂直间距
        legend.append("rect")
            .attr("x", 60)
            .attr("width", legend_size*2/3)
            .attr("height", legend_size*2/3)
            .attr("fill", d => d.color);
        legend.append("text")
            .attr("x", 60 + legend_size)
            .attr("y", legend_size*1/3)
            .attr("dy", '.35em')
            .text(d => d.name);
        this.legend = legend;
        
        const categories_hierarchy = [
            { name: "Level 1", color: 'rgb(255, 23, 23)' },
            { name: "Level 2", color: 'rgb(255, 139, 23)' },
            { name: "Level 3", color: 'rgb(255, 238, 23)' },
            { name: "Level 4", color: 'rgb(180, 255, 23)' },
            { name: "Level 5", color: 'rgb(23, 255, 35)' },
            { name: "Level 6", color: 'rgb(23, 255, 255)' },
            { name: "Level 7", color: 'rgb(23, 64, 255)' },
            { name: "Level 8", color: 'rgb(168, 23, 255)' },
            { name: "Level 9", color: 'rgb(255, 83, 247)' },
            { name: "…", color: 'transparent' },
        ];
        const legend_hierarchy = this.svg.selectAll(".legend2")
            .data(categories_hierarchy)
            .enter().append("g")
            .attr("class", "legend2")
            .attr("transform", (d, i) => `translate(0, ${i * legend_size})`); // 设定每个图例的垂直间距
        legend_hierarchy.append("rect")
            .attr("x", 60)
            .attr("width", legend_size*2/3)
            .attr("height", legend_size*2/3)
            .attr("fill", d => d.color);
        legend_hierarchy.append("text")
            .attr("x", 60 + legend_size)
            .attr("y", legend_size*1/3)
            .attr("dy", '.35em')
            .text(d => d.name);
        legend_hierarchy.attr("visibility", "hidden");
        this.legend_hierarchy = legend_hierarchy;
    },
};
</script>

<style scoped>

.svg-info {
    position: relative;
    height: 25%;
    width: 100%;
}

.tree-box {
    position: relative;
    border: 1px solid #ddd;
    width: 100%;
    height: 100%;
    flex-shrink: 100;
}

.datainfo-svg {
    width: 100%;
    height: 100%;
}

.datainfo-bar {
    height: 10%;
    width: 90%;
}

.meta-label {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.V-centered {
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.H-centered {
    align-items: center;
    text-align: center;
}

</style>
