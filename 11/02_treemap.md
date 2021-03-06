트리맵 Treemap
===

목표
---
- d3-hierarchy의 d3.treemap을 통해 treemap을 구현해본다


d3-hierarchy
---
http://devdocs.io/d3~4/d3-hierarchy#hierarchy
- 위계 구조를 가진 데이터를 처리하는 모듈

## d3.hierarchy
- 특정한 레이아웃에 데이터를 전달하기 전에 nested된 형태의 데이터를 d3.hierarchy에 전달하여 변형을 한 후, 특정한 레이아웃(treemap, tree 등)에 전달하여 변형한다.
- 결과로 개별 노드에는  depth, parent, children 등이 계산된 데이터가 계산된다.
  - node.data - 변형전 데이터
  - node.depth - 루트 노드는 0, 이후 단계마다 1씩 증가
  - node.height - 리프 노드는 0, 이후 단계마다 거꾸로 1씩 증가
  - node.parent - 부모 노드의 오브젝트, 루트 노드는 `undefined`
  - node.children - 자식 노드의 배열, 리프 노드는 `undefined`
  - node.value - node.sum 실행시 특정 값을 합산하여 가지고 있음(treemap 사용시 반드시 node.sum을 실행시킨 후 전달해야함)

- 테이블 형태의 자료의 경우(parent를 가리키는 field 있어야함) [d3.stratify](http://devdocs.io/d3~4/d3-hierarchy#stratify)를 이용해 parent-children 구조로 변형이 가능

- 이미 nested 된 형태의 데이터를 d3.hierarchy에 전달
```javascript
d3.json('flare.json', callback);
function callback(err, data) {
  if(err) return console.error(err);
  console.log(data);
  var hierarchy = d3.hierarchy(data, function(d){return d.children});
  console.log(hierarchy);
}
```

- `size`로 합산하고, `children`을 `height`과 `value`순으로 정렬
```javascript
hierarchy.sum(function(d){return d.size})
  .sort(function(a, b) { return b.height - a.height || b.value - a.value; }) // expr1 || expr2 일때 expr1이 falsy value면 다음 값을 뱉어냄
```


d3.Treemap
---

- 초기 설정
```javascript
var w = 600, h = 300;
var margin = {top:10, right:10, bottom: 10, left: 10};
var innerW = w - margin.right - margin.left,
  innerH = h - margin.top - margin.bottom;

var svg = d3.select('body').append('svg')
    .attr('width', w)
    .attr('height', h)
  .append('g')
    .attr('transform', 'translate('+ [margin.left, margin.top] + ')');

```

- 스케일 설정
 - 깊이depth가 1인 노드들의 이름이 개별 색상을 가리키고, 리프노드의 값value가 opacity를 결정하도록 설정
 - `node.leaves` 통해 말단 노드들에 접근  

```javascript
var opacityDomain = d3.extent(hierarchy.leaves(), function(d){return d.value;});
var opacity = d3.scaleLinear().domain(opacityDomain).range([0.4, 1.0]);
var colorDomain = hierarchy.children.map(function(d){return d.data.name});
var color = d3.scaleOrdinal().domain(colorDomain).range(d3.schemeCategory10);
```

- d3.treemap 통해서 데이터 변형

```javascript
var treemap = d3.treemap()
  .size([innerW, innerH]);
hierarchy = treemap(hierarchy);
```

- node.descendants를 통해 개별 노드를 모두 배열로 반환

```javascript
var nodes = hierarchy.descendants();
```


- 리프노드만 사각형 영역을 그린다.

```javascript
var leaf = svg.selectAll('.leaf')
  .data(nodes.filter(function(n){return !n.children}), function(d){return d.data.name})
  .enter().append('rect')
    .attr('class', 'leaf')
  .attr('x', function(d){return d.x0})
  .attr('y', function(d){return d.y0})
  .attr('width', function(d){return d.x1-d.x0})
  .attr('height', function(d){return d.y1-d.y0})
  .style('fill-opacity', function(d){return opacity(d.value)});
```


- 색상을 결정하기 위해선 깊이가 1인 부모 노드를 찾아 색상을 확인해야함
 - 아래와 같이 개별 특정 깊이의 부모를 찾는 함수를 정의

```javascript
function findParent(node, depth) {

      if(node.depth < depth) {
        return null;
      } else if(node.depth === depth) {
        return node;
      }
      if(node.parent) {
        return findParent(node.parent, depth); //재귀 recursion
      }
    }
```

```javascript
leaf.style('fill', function(d){return color(findParent(d, 1).data.name)});
```
