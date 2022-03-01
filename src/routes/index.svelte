<script lang="ts">
  /*
    --Configs--
    Dark mode
    number of nodes
    % of edges
    visualization speed
    regenerate graph

    Then click on startnode to run the algorithm
    Start node is red and maybe a bit bigger
    Edges are grey, black once selected for the MST
    Current working edge might be red?
  */

  //Generate nodes (in rust)
  //Generate edges (shorter edges more likely to be created) (also in rust)

  //Select start node in js, then run the algorithm in rust.
  //That returns a sorted array with all edges belonging to the mst
  //The animation is then in pure JS.
  import { onMount } from 'svelte'
  import Node from '../components/Node.svelte'

  interface Node {
    x: number,
    y: number
  }

  interface Edge {
    n1: Node,
    n2: Node
  }

  let windowWidth: number = 100;
  let windowHeight: number = 100;

  const generateNodes = (n: number): Node[] => {
    let nodes: Node[] = []
    for (let i=0; i<=n; i++) {
      nodes.push({
        x: Math.round(Math.random()*windowWidth),
        y: Math.round(Math.random()*windowHeight)
      })
    }
    return nodes
  }

  const generateEdges = (n: number): Edge[] => {
    let edges: Edge[] = []
    for (let i=0; i<n; i++) {
      edges.push({
        n1: nodes[Math.floor((Math.random()*nodes.length))],
        n2: nodes[Math.floor((Math.random()*nodes.length))]
      })
    }
    return edges.filter((edge) => edge.n1 != edge.n2) //Returns all edges that don't loop on themselves
  }

  const drawEdges = (edges: Edge[]): void => {
    const lc: HTMLCanvasElement = document.querySelector('#lineCanvas')
    const c: CanvasRenderingContext2D = lc.getContext('2d')
    edges.forEach(e => {
      //c.lineWidth = 2
      //c.fillStyle = '#aaa'
      c.beginPath()
      c.moveTo(e.n1.x, e.n1.y)
      c.lineTo(e.n2.x, e.n2.y)
      c.stroke()
    })
  }

  let nodes: Node[] = []
  let edges: Edge[] = []
  onMount(() => {
    const nodesCount: number = 150 
    nodes = generateNodes(nodesCount)
    edges = generateEdges(nodesCount*0.5)
    drawEdges(edges)
  })
</script>

<div class="main" bind:clientWidth={windowWidth} bind:clientHeight={windowHeight}>
  <!-- width={windowWidth} height={windowHeight} -->
  <canvas id="lineCanvas" width="1400" height="600"></canvas>
  {#each nodes as node}
    <Node x={node.x} y={node.y}/>
  {/each}
</div>

<style>
  .main {
    position: relative;
    width: 97vw;
    height: 94vh;
  }
  #lineCanvas {
    position: absolute;
    background-color: green;
    left: 1rem;
    top: 1rem;
  }
</style>