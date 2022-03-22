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

  interface EdgeWithLength {
    edge: Edge,
    length: number
  }

  const dst = (n1: Node, n2: Node): number => {
    return Math.sqrt(Math.pow((n1.x-n2.x),2) + Math.pow((n1.y-n2.y),2))
  }

  let windowWidth: number
  let windowHeight: number

  const generateNodes = (n: number): Node[] => {
    let nodes: Node[] = []
    for (let i = 0; i < n; i++) {
      nodes.push({
        x: Math.round( Math.random()*(windowWidth*0.98) + (windowWidth*0.01) ),
        y: Math.round( Math.random()*(windowHeight*0.96) + (windowHeight*0.02) )
      })
    }
    return nodes
  }

  const generateEdges = (): Edge[] => {

    //Construct all unique edges and calculate their length
    let allEdges: EdgeWithLength[] = []
    for (let i = 0; i < NODES.length - 1; i++) {
      for (let j = i+1; j < NODES.length; j++) {
        allEdges.push({
          edge: {
            n1: NODES[i],
            n2: NODES[j]
          },
          length: dst(NODES[i], NODES[j])
        })
      }
    }

    //Sorts all edges for each node and then selects some of the shortest
    //edges to actually be sent back to the renderer
    let edges: Edge[] = []
    for (let i = 0; i < NODES.length; i++) {
      const n = NODES[i]
      const edgesToCurrentNode: EdgeWithLength[] = allEdges.filter(e => e.edge.n1 == n || e.edge.n2 == n)
      const orderedEdges = edgesToCurrentNode.sort( (a, b) => (a.length > b.length) ? 1 : -1 )
      
      const transferEdgeBelowIndex = (a: number): void => {
        edges.push( orderedEdges.splice(Math.floor(Math.random()*a),1).at(0).edge )
      }

      transferEdgeBelowIndex(2)
      transferEdgeBelowIndex(4)
      transferEdgeBelowIndex(8)
      if(0.2 > Math.random()) transferEdgeBelowIndex(40)
    }

    return edges
  }

  const drawEdges = (edges: Edge[], delay: number): void => {
    const lc: HTMLCanvasElement = document.querySelector('#lineCanvas')

    lc.style.width = '100%';
    lc.style.height = '100%';
    lc.width = lc.offsetWidth;
    lc.height = lc.offsetHeight;

    const c: CanvasRenderingContext2D = lc.getContext('2d')

    if (delay === 0) {
      edges.forEach( e => {
        c.lineWidth = 3
        c.strokeStyle = '#ccc'
        c.beginPath()
        c.moveTo(e.n1.x, e.n1.y)
        c.lineTo(e.n2.x, e.n2.y)
        c.stroke()
      })
    }

    let edgeIndex: number = 0
    let drawEdgeInterval = setInterval(() => {
      c.lineWidth = 3
      c.strokeStyle = '#ccc'
      c.beginPath()
      c.moveTo(edges[edgeIndex].n1.x, edges[edgeIndex].n1.y)
      c.lineTo(edges[edgeIndex].n2.x, edges[edgeIndex].n2.y)
      c.stroke()
      edgeIndex++
      if (edgeIndex >= edges.length) clearInterval(drawEdgeInterval)
    }, delay)


  }

  const primsAlgorithm = (edges: Edge[], n: Node): Edge[] => {

    const sameNode = (n1: Node, n2: Node): boolean => {
      return (n1.x === n2.x && n1.y === n2.y)
    }

    const getEdgeWithLength = (edge: Edge): EdgeWithLength => {
      return {
        edge: edge,
        length: dst(edge.n1, edge.n2)
      }
    }

    const getShortestEdge = (edges: EdgeWithLength[]): Edge => {
      return edges.sort( (a, b) => (a.length > b.length) ? 1 : -1 ).at(0).edge 
    }

    const getShortestEdgeFromNode = (edges: Edge[], node: Node): Edge => {
      let reachableEdgesWithLengths: EdgeWithLength[] = edges.filter(edge => (sameNode(edge.n1, n) || sameNode(edge.n2, n))).map(getEdgeWithLength)
      return getShortestEdge(reachableEdgesWithLengths)
    }

    const getAllNodesFromEdges = (edges: Edge[]): Set<Node> => {
      let allNodes = new Set<Node>()
      edges.forEach(edge => allNodes.add(edge.n1).add(edge.n2)) //Adds both edges
      return allNodes
    }

    const getReachableEdges = (edges: Edge[], visitedNodes: Node[]): Edge[] => {
      let reachableEdges: Edge[] = []
      edges.forEach( edge => {
        if ( !((visitedNodes.includes(edge.n1) && visitedNodes.includes(edge.n2)) || (!visitedNodes.includes(edge.n1) && !visitedNodes.includes(edge.n2))) ){
          reachableEdges.push(edge)
        }
      })
      return reachableEdges
    }


    let mstEdges: Edge[] = [getShortestEdgeFromNode(edges, n)]
    const nodeCount: number = getAllNodesFromEdges(edges).size
    while ( nodeCount > getAllNodesFromEdges(mstEdges).size ) {

      let visitedNodes: Node[] = [...getAllNodesFromEdges(mstEdges)]

      //Find the all reachable edges and calculates their lengths
      let reachableEdges: EdgeWithLength[] = getReachableEdges(edges, visitedNodes).map(getEdgeWithLength)
      
      //Push the shortest edge to the spanning tree list
      mstEdges.push(getShortestEdge(reachableEdges))
    }

    return mstEdges
  }

  const startVisualization = (n: Node): void => {
    drawEdges(primsAlgorithm(EDGES, n), 20)
  }

  let NODES: Node[] = []
  let EDGES: Edge[] = []
  onMount(() => {
    const nodesCount: number = 200 
    NODES = generateNodes(nodesCount)
    EDGES = generateEdges()
    drawEdges(EDGES, 0)
  })
</script>

<div class="main" bind:clientWidth={windowWidth} bind:clientHeight={windowHeight}>
  <!-- width={windowWidth} height={windowHeight} -->
  <canvas id="lineCanvas"></canvas>
  {#each NODES as node}
    <Node x={node.x} y={node.y} on:click={() => startVisualization(node)}/>
  {/each}
</div>

<style>
  .main {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
  #lineCanvas {
    position: absolute;
    background-color: white;
  }
</style>