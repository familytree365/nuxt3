<template>
  <div>
    <loading
      :active="false"
      :color="color"
      :background-color="backgroundColor"
    />

    <v-select
      label="name"
      v-model="selected_option"
      :reduce="person => person.id"
      :options="persons"
      @input="setSelected"
    />

    <div id="tree"></div>

    <button id='saveButton'>Export my PNG</button>
  </div>
</template>
<router>
{
  name: 'trees.show'
}
</router>
<script>
import * as d3Base from 'd3'
import {coordQuad, dagConnect, dagStratify, decrossOpt, layeringSimplex, sugiyama} from 'd3-dag'
import d3Tip from 'd3-tip'
import vSelect from 'vue-select'
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'
import FileSaver  from 'file-saver';
import { ref, computed, useStore } from 'vue';
const d3 = Object.assign({}, d3Base, {tip: d3Tip()})

Array.prototype.remove = function () {
  let what, a = arguments, L = a.length, ax

  while (L && this.length) {
    what = a[--L];

    while ((ax = this.indexOf(what)) !== -1) {
      this.splice(ax, 1)
    }
  }

  return this
};

export default {
  layout: 'auth',
  components: {
    vSelect,
    Loading
  },
  head: {
    title: "Pedigree Tree View"
  },
  setup() {
    const person = ref([])
    const selected_option = ref(null)
    const data = ref({})
    const all_nodes = ref([])
    const tree = ref(null)
    const dag = ref(null)
    const svg = ref(null)
    const g = ref(null)
    const tip = ref(null)
    const duration = ref(750)
    const zoom = ref(null)
    const isLoading = ref(true)
    const fullPage = ref(true)
    const color = ref('#4fcf8d')
    const backgroundColor = ref('#ffffff')
    created(() => {
      this.persons = await this.$axios.$get("/api/get_person");
    })
    async function setSelected(value) {
      this.isLoading = true

      const params = {
        start_id: value,
      }

      try {
        let gedComdata = await this.$axios
          .$get("/api/trees/show", {params})
        this.data = gedComdata
        if(!this.data.links.length) {
          console.log('no data found');
          return;
        }
        this.data.links = this.data.links
          .map((item) => item.map(childItem => childItem.toString()))

        d3.selectAll("#tree")
          .selectAll('svg').remove()

        this.isLoading = false
        // this.generate();
        setTimeout(this.generate, 1000)
      } catch (error) {
        console.log(error)
      }

      this.isLoading = false
    }
    async function fetchdata() {
      this.isLoading = true

      try {
        this.data = await this.$axios.$get("/api/trees/show")
        this.isLoading = false
        this.generate()
      } catch {
        console.log(error)
      }

      this.isLoading = false
    }
    function generate() {
      Object.keys(this.data.unions)
        .forEach(key => this.data.unions[key].isUnion = true)
      Object.keys(this.data.persons)
        .forEach(key => this.data.persons[key].isUnion = false)
      let margin = {top: 20, right: 0, bottom: 30, left: 0},
        width = 1500 - margin.left - margin.right,
        height = 800 - margin.top - margin.bottom
      let screen_width = width,
        screen_height = height
      this.zoom = d3.zoom()
        .on("zoom", (event, d) => this.g.attr("transform", event.transform))
      this.tip = d3.tip
        .attr('class', 'd3-tip')
        .direction('e')
        .offset([0, 5])
        .html((d) => {
          if (d.data.isUnion) {
            return
          }
          let content = `
              <span style='margin-left: 2.5px;'><b>` + d.data.name + `</b></span><br>
                <table style="margin-top: 2.5px;">
                  <tr><td>born</td><td>` + (this.getFullDisplayBirthYear(d) || "?") + ` in ` + (d.data.birthday_plac || "?") + `</td></tr>
                  <tr><td>died</td><td>` + (this.getFullDisplayDeathYear(d) || "?") + ` in ` + (d.data.deathday_plac || "?") + `</td></tr>
                </table>`

          return content.replace(new RegExp("null", "g"), "?")
        })
      this.svg = d3.select("#tree").append("svg")
        .attr("width", screen_width)
        .attr("height", screen_height)
        .call(this.zoom)
        .call(this.tip)
      this.g = this.svg.append("g");
      let i = 0,
        x_sep = 120,
        y_sep = 50;
      try {
        this.tree = sugiyama()
          .nodeSize(() => [y_sep, x_sep])
          .layering(layeringSimplex())
          .decross(decrossOpt())
          .coord(coordQuad());
      } catch (e) {
        console.log(e)
      }
      this.dag = dagConnect()(this.data.links)
      if (!!this.dag.id) {
        let root = this.dag.copy()
        root.id = undefined
        this.dag = root
      }
      this.all_nodes = this.dag.descendants()
      this.all_nodes.forEach(n => {
        n.data = this.data.persons[n.data.id] ? this.data.persons[n.data.id] : this.data.unions[n.data.id]
        n.childrens = n.children() // all nodes collapsed by default
        n._children = n.children() // all nodes collapsed by default
        n.inserted_nodes = []
        n.inserted_roots = []
        n.neighbors = []
        n.visible = false
        n.inserted_connections = []
      })
      let root = this.all_nodes.find((n) => {
        if(n.data){return n.data.id === this.data.start}})
      root.visible = true
      root.neighbors = []
      root.neighbors = this.getNeighbors(root)
      root.x0 = screen_height / 2
      root.y0 = screen_width / 2
      this.update(root)
    }
    function remove_inserted_root_nodes(n) {
      n.inserted_connections.forEach(
        arr => {
          if (arr[0].childrens.includes(arr[1])) {
            arr[0]._children.push(arr[1])
            arr[0].childrens.remove(arr[1])
          }
        }
      )
      n.inserted_nodes.forEach(this.remove_inserted_root_nodes);
    }
    function collapse(d) {
      this.remove_inserted_root_nodes(d);
      let vis_inserted_neighbors = d.neighbors.filter(n => n.visible && d.inserted_nodes.includes(n));
      vis_inserted_neighbors.forEach(
        n => {
          n.visible = false;
          if (d.childrens.includes(n)) {
            d._children.push(n);
            d.childrens.remove(n);
          }
          if (n.childrens.includes(d)) {
            n._children.push(d);
            n.childrens.remove(d);
          }
          if (n.data.isUnion) {
            this.collapse(n);
          }
          d.inserted_nodes.remove(n);
        }
      );
    }
    function add_root_nodes(n) {
      n.inserted_connections.forEach(
        arr => {
          if (arr[0]._children.includes(arr[1])) {
            arr[0].childrens.push(arr[1]);
            arr[0]._children.remove(arr[1]);
          }
        }
      )
      n.inserted_nodes.forEach(this.add_root_nodes)
    }
    function uncollapse(d, make_roots) {
      if (d === undefined) return;
      let extended_neighbors = d.neighbors.filter(n => n.visible)
      extended_neighbors.forEach(
        n => {
          if (d._children.includes(n)) {
            d.inserted_connections.push([d, n]);
          }
          if (n._children.includes(d)) {
            d.inserted_connections.push([n, d]);
          }
        }
      )
      let collapsed_neighbors = d.neighbors.filter(n => !n.visible);
      collapsed_neighbors.forEach(
        n => {
          n.neighbors = this.getNeighbors(n);
          n.visible = true;
          if (d._children.includes(n)) {
            d.childrens.push(n);
            d._children.remove(n);
          }
          if (n._children.includes(d)) {
            n.childrens.push(d);
            n._children.remove(d);
            if (make_roots && !d.inserted_roots.includes(n)) {
              d.inserted_roots.push(n);
            }
          }
          if (n.data.isUnion) {
            this.uncollapse(n, true);
          }
          d.inserted_nodes.push(n);
        }
      )
      if (!make_roots) {
        this.add_root_nodes(d);
      }
    }
    function is_extendable(node) {
      return node.neighbors.filter(n => !n.visible).length > 0
    }
    function getNeighbors(node) {
      if (node.data.isUnion) {
        return this.getChildren(node)
          .concat(this.getPartners(node))
      } else {
        return this.getOwnUnions(node)
          .concat(this.getParentUnions(node))
      }
    }
    function getParentUnions(node) {
      if (node === undefined) return [];
      if (node.data.isUnion) return [];
      let u_id = node.data.parent_union;
      if (u_id) {
        let union = this.all_nodes.find((n) => {
        if(n.data){return n.data.id === u_id}})
        return [union].filter(u => u !== undefined);
      } else return [];
    }
    function getParents(node) {
      let parents = [];
      if (node.data.isUnion) {
        node.data.partner.forEach(
          p_id => parents.push(this.all_nodes.find(n => n.data.id === p_id))
        );
      } else {
        let parent_unions = this.getParentUnions(node);
        parent_unions.forEach(
          u => parents = parents.concat(this.getParents(u))
        );
      }
      return parents.filter(p => p !== undefined)
    }
    function getOtherPartner(node, union_data) {
      let partner_id = union_data.partner.find(
        p_id => p_id !== node.data.id && p_id !== undefined
      );
      return this.all_nodes.find(n => n.data.id === partner_id)
    }
    function getPartners(node) {
      let partners = [];
      if (node.data.isUnion) {
        node.data.partner.forEach(
          p_id => partners.push(this.all_nodes.find(n => n.data.id === p_id))
        )
      }
      else {
        let own_unions = this.getOwnUnions(node);
        own_unions.forEach(
          u => {
            partners.push(this.getOtherPartner(node, u.data))
          }
        )
      }
      return partners.filter(p => p !== undefined)
    }
    function getOwnUnions(node) {
      if (node.data.isUnion) return [];
      let unions = [];
      node.data.own_unions.forEach(
        u_id => unions.push(this.all_nodes.find(n => n.data.id === u_id))
      );
      return unions.filter(u => u !== undefined)
    }
    function getChildren(node) {
      let children = [];
      if (node.data.isUnion) {
        children = node.childrens.concat(node._children);
      } else {
        let own_unions = this.getOwnUnions(node);
        own_unions.forEach(
          u => children = children.concat(this.getChildren(u))
        )
      }
      children = children
        .filter(c => c !== undefined)
        .sort((a, b) => Math.sign((this.getFullDisplayBirthYear(a) || 0) - (this.getFullDisplayDeathYear(b) || 0)));
      return children
    }
    function getFullDisplayBirthYear(node) {
      let birthYear = ''
      if (node.data && node.data.birth_year) {
        return node.data.birth_year
      }

      if (node.data && node.data.birthday) {
        return this.getBirthYear(node)
      }
    }
    function getFullDisplayDeathYear(node) {
      let deathYear = ''
      if (node.data && node.data.death_year) {
        return node.data.death_year
      }

      if (node.data && node.data.deathday) {
        return this.getDeathYear(node)
      }
    }
    function getBirthYear(node) {
      return new Date(node.data.birthday || NaN).getFullYear()
    }
    function getDeathYear(node) {
      return new Date(node.data.deathday || NaN).getFullYear()
    }
    function find_path(n) {
      let parents = this.getParents(n);
      let found = false;
      let result = null;
      parents.forEach(
        p => {
          if (p && !found) {
            if (p.id === "profile-89285291") {
              found = true;
              result = [p, n];
            } else {
              result = this.find_path(p);
              if (result) {
                found = true;
                result.push(n)
              }
            }
          }
        }
      )
      return result
    }
    function update(source) {
      if (source.data.id !== this.data.start) return false;
      let dag_tree = this.tree(this.dag);
      let nodes = this.dag.descendants();
      let links = this.dag.links();
      let node = this.g.selectAll('g.node')
        .data(nodes, (d) => {
          if(d.data){return d.data.id || (d.data.id = ++i);}
        })
      let nodeEnter = node.enter().append('g')
        .attr('class', 'node')
        .attr("transform", (d) => {
          return "translate(" + source.y0 + "," + source.x0 + ")";
        })
        .on('click', this.onClick)
        .on('mouseover', (e, d) => this.tip.show(d, e.target))
        .on('mouseout', (e, d) => this.tip.hide(d, e.target))
        .attr('visibility', 'visible');
      nodeEnter.append('circle')
        .attr('class', 'node')
        .attr('r', 1e-6)
        .style("fill", (d) => {
          return this.is_extendable(d) ? "lightsteelblue" : "#fff";
        });
      nodeEnter.append('text')
        .attr("dy", "-2")
        .attr("x", 13)
        .attr("text-anchor", "start")
        .text((d) => {if(d.data){return d.data.name}});
      nodeEnter.append('text')
        .attr("dy", "10")
        .attr("x", 13)
        .attr("text-anchor", "start")
        .text(
          (d) => {
            if (d.data && d.data.isUnion) return;
            return (this.getFullDisplayBirthYear(d) || "?") + 
              " - " +
              (this.getFullDisplayDeathYear(d) || "?")
          }
        );
      let nodeUpdate = nodeEnter.merge(node);
      nodeUpdate.transition()
        .duration(this.duration)
        .attr("transform", (d) => {
          return "translate(" + d.y + "," + d.x + ")";
        });
      nodeUpdate.select('circle.node')
        .attr('r', (d) => {if(d.data){10 * !d.data.isUnion}})
        .style("fill", (d) => {
          return this.is_extendable(d) ? "lightsteelblue" : "#fff";
        })
        .attr('cursor', 'pointer');
      let nodeExit = node.exit().transition()
        .duration(this.duration)
        .attr("transform", (d) => {
          return "translate(" + source.y + "," + source.x + ")";
        })
        .attr('visibility', 'hidden')
        .remove();
      nodeExit.select('circle')
        .attr('r', 1e-6);
      nodeExit.select('text')
        .style('fill-opacity', 1e-6);
      let link = this.g.selectAll('path.link')
        .data(links, (d) => {
          return d.source.id + d.target.id
        });
      let linkEnter = link.enter().insert('path', "g")
        .attr("class", "link")
        .attr('d', (d) => {
          let o = {x: source.x0, y: source.y0}
          return this.diagonal(o, o)
        });
      let linkUpdate = linkEnter.merge(link);
      linkUpdate.transition()
        .duration(this.duration)
        .attr('d', d => this.diagonal(d.source, d.target));
      let linkExit = link.exit().transition()
        .duration(this.duration)
        .attr('d', (d) => {
          let o = {x: source.x, y: source.y}
          return this.diagonal(o, o)
        })
        .remove();
      this.svg.transition()
        .duration(this.duration)
        .call(
          this.zoom.transform,
          d3.zoomTransform(this.g.node()).translate(-(source.y - source.y0), -(source.x - source.x0)),
        );
      nodes.forEach((d) => {
        d.x0 = d.x;
        d.y0 = d.y;
      });
      d3.select('#saveButton').on('click', () => {
        let svgString = this.getSVGString(this.svg.node());
        this.svgString2Image(
          svgString,
          this.svg.node().getBBox().width,
          this.svg.node().getBBox().height,
          'png',
          this.onSave
        ); // passes Blob and filesize String to the callback
      });
    }
    function diagonal(s, d) {
       let path = `M ${s.y} ${s.x}
        C ${(s.y + d.y) / 2} ${s.x},
          ${(s.y + d.y) / 2} ${d.x},
          ${d.y} ${d.x}`
      return path
    }
    function getCSSStyles(parentElement) {
      let selectorTextArr = [];
      selectorTextArr.push('#' + parentElement.id);
      for (let c = 0; c < parentElement.classList.length; c++)
        if (!contains('.' + parentElement.classList[c], selectorTextArr))
          selectorTextArr.push('.' + parentElement.classList[c]);
      let nodes = parentElement.getElementsByTagName("*");
      for (let i = 0; i < nodes.length; i++) {
        let id = nodes[i].id;
        if (!contains('#' + id, selectorTextArr))
          selectorTextArr.push('#' + id);
        let classes = nodes[i].classList;
        for (let c = 0; c < classes.length; c++)
          if (!contains('.' + classes[c], selectorTextArr))
            selectorTextArr.push('.' + classes[c]);
      }
      let extractedCSSText = "";
      for (let i = 0; i < document.styleSheets.length; i++) {
        let s = document.styleSheets[i];
        try {
          if (!s.cssRules) continue;
        } catch (e) {
          if (e.name !== 'SecurityError') throw e; // for Firefox
          continue;
        }
        let cssRules = s.cssRules;
        for (let r = 0; r < cssRules.length; r++) {
          if (contains(cssRules[r].selectorText, selectorTextArr))
            extractedCSSText += cssRules[r].cssText;
        }
      }
      return extractedCSSText;
      function contains(str, arr) {
        return arr.indexOf(str) === -1 ? false : true;
      }
    }
    function appendCSS(cssText, element) {
      let styleElement = document.createElement("style");
      styleElement.setAttribute("type", "text/css");
      styleElement.innerHTML = cssText;
      let refNode = element.hasChildNodes() ? element.children[0] : null;
      element.insertBefore(styleElement, refNode);
    }
    function getSVGString(svgNode) {
      svgNode.setAttribute('xlink', 'http://www.w3.org/1999/xlink');
      let cssStyleText = this.getCSSStyles(svgNode);
      this.appendCSS(cssStyleText, svgNode);
      let serializer = new XMLSerializer();
      let svgString = serializer.serializeToString(svgNode);
      svgString = svgString.replace(/(\w+)?:?xlink=/g, 'xmlns:xlink='); // Fix root xlink without namespace
      svgString = svgString.replace(/NS\d+:href/g, 'xlink:href'); // Safari NS namespace fix
      return svgString;
    }
    function svgString2Image(svgString, width, height, format, callback) {
      format = format ? format : 'png';
      let imgsrc = 'data:image/svg+xml;base64,' + btoa(unescape(encodeURIComponent(svgString))); // Convert SVG string to data URL
      let canvas = document.createElement("canvas");
      let context = canvas.getContext("2d");
      canvas.width = width;
      canvas.height = height;
      let image = new Image();
      image.onload = () => {
        context.clearRect(0, 0, width, height);
        context.drawImage(image, 0, 0, width, height);
        canvas.toBlob((blob) => {
          let filesize = Math.round(blob.length / 1024) + ' KB';
          if (callback) callback(blob, filesize);
        });
      };
      image.src = imgsrc;
    }
    function onClick(e, d) {
      if (d.data.isUnion) return;
      if (this.is_extendable(d)) this.uncollapse(d)
      else this.collapse(d)
      this.update(d);
    }
    function onSave(dataBlob, filesize) {
      FileSaver.saveAs(dataBlob, 'D3 vis exported to PNG.png'); // FileSaver.js function
    }
  }
}
</script>

<style>
.node circle {
  fill: #fff;
  stroke: steelblue;
  stroke-width: 3px;
}

.node text {
  font: 12px sans-serif;
}

.link {
  fill: none;
  stroke: #ccc;
  stroke-width: 2px;
}

.d3-tip {
  font: 12px sans-serif;
  padding: 12px;
  background: rgba(0, 0, 0, 0.8);
  color: #fff;
  border-radius: 2px;
}

#saveButton {
  border: 1px solid;
  padding: 5px;
  border-radius: 3px;
  background-color: #4fcf8d;
  color: white;
}
</style>
