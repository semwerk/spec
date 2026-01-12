# D3 Force-Directed Graph Examples

**Version:** 1.0.0
**Status:** Draft

## Overview

Examples of rendering Semwerk journeys and concept graphs as D3.js force-directed graphs.

## Journey Graph Data Format

Convert journey YAML to D3 graph format:

### TypeScript Converter

```typescript
import { Journey } from '@semwerk/werkspec';

interface D3Node {
  id: string;
  name: string;
  type: 'stage' | 'milestone' | 'decision' | 'jump_off';
  group: number;
}

interface D3Link {
  source: string;
  target: string;
  label?: string;
  condition?: string;
  type: 'internal' | 'external';
}

interface D3Graph {
  nodes: D3Node[];
  links: D3Link[];
}

function journeyToD3(journey: Journey): D3Graph {
  const nodes: D3Node[] = journey.nodes.map((node, index) => ({
    id: node.id,
    name: node.name,
    type: node.type,
    group: getGroupForType(node.type),
  }));

  const links: D3Link[] = [];
  for (const node of journey.nodes) {
    for (const conn of node.connections) {
      if (conn.target_node_id) {
        links.push({
          source: node.id,
          target: conn.target_node_id,
          label: conn.label,
          condition: conn.condition,
          type: 'internal',
        });
      }
      if (conn.target_journey) {
        links.push({
          source: node.id,
          target: conn.target_journey,
          label: conn.label,
          type: 'external',
        });
      }
    }
  }

  return { nodes, links };
}

function getGroupForType(type: string): number {
  switch (type) {
    case 'stage': return 1;
    case 'milestone': return 2;
    case 'decision': return 3;
    case 'jump_off': return 4;
    default: return 0;
  }
}
```

## D3 Rendering

```javascript
// Load and render journey
fetch('journey.yaml')
  .then(res => res.text())
  .then(yaml => {
    const journey = parseJourney(yaml);
    const graphData = journeyToD3(journey);
    renderD3Graph(graphData);
  });

function renderD3Graph(data) {
  const width = 800;
  const height = 600;

  const svg = d3.select('#graph')
    .append('svg')
    .attr('width', width)
    .attr('height', height);

  // Force simulation
  const simulation = d3.forceSimulation(data.nodes)
    .force('link', d3.forceLink(data.links).id(d => d.id).distance(100))
    .force('charge', d3.forceManyBody().strength(-300))
    .force('center', d3.forceCenter(width / 2, height / 2));

  // Links
  const link = svg.append('g')
    .selectAll('line')
    .data(data.links)
    .enter().append('line')
    .attr('stroke', d => d.type === 'external' ? '#999' : '#666')
    .attr('stroke-width', 2)
    .attr('stroke-dasharray', d => d.type === 'external' ? '5,5' : '0');

  // Link labels
  const linkLabel = svg.append('g')
    .selectAll('text')
    .data(data.links.filter(d => d.label))
    .enter().append('text')
    .attr('font-size', 10)
    .attr('fill', '#666')
    .text(d => d.label);

  // Nodes
  const node = svg.append('g')
    .selectAll('circle')
    .data(data.nodes)
    .enter().append('circle')
    .attr('r', d => getNodeRadius(d.type))
    .attr('fill', d => getNodeColor(d.type))
    .call(d3.drag()
      .on('start', dragstarted)
      .on('drag', dragged)
      .on('end', dragended));

  // Node labels
  const label = svg.append('g')
    .selectAll('text')
    .data(data.nodes)
    .enter().append('text')
    .attr('text-anchor', 'middle')
    .attr('dy', 4)
    .attr('font-size', 12)
    .attr('fill', '#fff')
    .text(d => d.name);

  // Update positions
  simulation.on('tick', () => {
    link
      .attr('x1', d => d.source.x)
      .attr('y1', d => d.source.y)
      .attr('x2', d => d.target.x)
      .attr('y2', d => d.target.y);

    linkLabel
      .attr('x', d => (d.source.x + d.target.x) / 2)
      .attr('y', d => (d.source.y + d.target.y) / 2);

    node
      .attr('cx', d => d.x)
      .attr('cy', d => d.y);

    label
      .attr('x', d => d.x)
      .attr('y', d => d.y);
  });

  function getNodeRadius(type) {
    switch (type) {
      case 'milestone': return 20;
      case 'decision': return 25;
      case 'jump_off': return 22;
      default: return 18;
    }
  }

  function getNodeColor(type) {
    switch (type) {
      case 'stage': return '#4A90E2';
      case 'milestone': return '#50C878';
      case 'decision': return '#FF6B6B';
      case 'jump_off': return '#9B59B6';
      default: return '#95A5A6';
    }
  }

  function dragstarted(event, d) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  }

  function dragged(event, d) {
    d.fx = event.x;
    d.fy = event.y;
  }

  function dragended(event, d) {
    if (!event.active) simulation.alphaTarget(0);
    d.fx = null;
    d.fy = null;
  }
}
```

## Concept Graph Rendering

```javascript
function conceptsToD3(graph) {
  const nodes = graph.concepts.map(concept => ({
    id: concept.id,
    name: concept.name,
    status: concept.status,
  }));

  const links = graph.relationships.map(rel => ({
    source: rel.from.replace('concept:@', ''),
    target: rel.to.replace('concept:@', ''),
    kind: rel.kind,
    weight: rel.weight,
  }));

  return { nodes, links };
}

function renderConceptGraph(data) {
  // Similar D3 setup with different styling
  const node = svg.selectAll('circle')
    .data(data.nodes)
    .enter().append('circle')
    .attr('r', 15)
    .attr('fill', d => d.status === 'deprecated' ? '#95A5A6' : '#3498DB');

  const link = svg.selectAll('line')
    .data(data.links)
    .enter().append('line')
    .attr('stroke', d => getLinkColor(d.kind))
    .attr('stroke-width', d => d.weight * 3)
    .attr('stroke-dasharray', d => d.kind === 'related' ? '5,5' : '0');

  function getLinkColor(kind) {
    switch (kind) {
      case 'parent': return '#2C3E50';
      case 'implements': return '#27AE60';
      case 'depends_on': return '#E74C3C';
      case 'documents': return '#F39C12';
      case 'related': return '#95A5A6';
      default: return '#7F8C8D';
    }
  }
}
```

## Interactive Features

```javascript
// Add tooltips
node.append('title')
  .text(d => `${d.name}\nType: ${d.type}`);

// Highlight on hover
node.on('mouseover', function(event, d) {
  d3.select(this).attr('r', 25);

  // Highlight connected nodes
  const connectedNodes = data.links
    .filter(l => l.source.id === d.id || l.target.id === d.id)
    .flatMap(l => [l.source.id, l.target.id]);

  node.attr('opacity', n => connectedNodes.includes(n.id) ? 1 : 0.3);
});

node.on('mouseout', function() {
  d3.select(this).attr('r', d => getNodeRadius(d.type));
  node.attr('opacity', 1);
});

// Click to navigate
node.on('click', function(event, d) {
  if (d.type === 'jump_off') {
    const target = d.connections[0].target_journey;
    loadJourney(target);
  }
});
```

## See Also

- [Mermaid Examples](./mermaid-examples.md) - Mermaid diagram generation
- [Journey Graph](../journey-graph.md) - Journey specification
- [Concept Graph](../../concepts/concept-graph.md) - Concept specification
