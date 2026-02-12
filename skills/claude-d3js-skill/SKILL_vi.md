---
name: d3-viz
description: Tạo trực quan hóa dữ liệu tương tác bằng d3.js. Kỹ năng này nên được sử dụng khi tạo biểu đồ tùy chỉnh, đồ thị, sơ đồ mạng, trực quan hóa địa lý hoặc bất kỳ trực quan hóa dữ liệu dựa trên SVG phức tạp nào yêu cầu kiểm soát chi tiết các yếu tố hình ảnh, chuyển đổi hoặc tương tác. Sử dụng kỹ năng này cho các trực quan hóa riêng biệt ngoài các thư viện biểu đồ tiêu chuẩn, cho dù trong React, Vue, Svelte, vanilla JavaScript hoặc bất kỳ môi trường nào khác.
---

# Trực quan hóa D3.js

## Tổng quan

Kỹ năng này cung cấp hướng dẫn để tạo các trực quan hóa dữ liệu tinh vi, tương tác bằng cách sử dụng d3.js. D3.js (Data-Driven Documents) vượt trội trong việc liên kết dữ liệu với các phần tử DOM và áp dụng các chuyển đổi dựa trên dữ liệu để tạo ra các trực quan hóa tùy chỉnh, chất lượng xuất bản với khả năng kiểm soát chính xác đối với mọi yếu tố hình ảnh. Các kỹ thuật này hoạt động trên mọi môi trường JavaScript, bao gồm vanilla JavaScript, React, Vue, Svelte và các framework khác.

## Khi nào sử dụng d3.js

**Sử dụng d3.js cho:**

- Các trực quan hóa tùy chỉnh yêu cầu mã hóa hình ảnh hoặc bố cục độc đáo
- Khám phá tương tác với các hành vi xoay, thu phóng hoặc quét phức tạp
- Trực quan hóa mạng/đồ thị (bố cục hướng lực, sơ đồ cây, phân cấp, biểu đồ dây)
- Trực quan hóa địa lý với các phép chiếu tùy chỉnh
- Trực quan hóa yêu cầu chuyển đổi mượt mà, được dàn dựng
- Đồ họa chất lượng xuất bản với khả năng kiểm soát kiểu dáng chi tiết
- Các loại biểu đồ mới lạ không có sẵn trong các thư viện tiêu chuẩn

**Cân nhắc các lựa chọn thay thế cho:**

- Trực quan hóa 3D - sử dụng Three.js thay thế

## Quy trình làm việc cốt lõi

### 1. Thiết lập d3.js

Nhập d3 ở đầu tập lệnh của bạn:

```javascript
import * as d3 from "d3";
```

Hoặc sử dụng phiên bản CDN (7.x):

```html
<script src="https://d3js.org/d3.v7.min.js"></script>
```

Tất cả các mô-đun (thang đo, trục, hình dạng, chuyển đổi, v.v.) đều có thể truy cập thông qua không gian tên `d3`.

### 2. Chọn mẫu tích hợp

**Mẫu A: Thao tác DOM trực tiếp (được khuyến nghị cho hầu hết các trường hợp)**
Sử dụng d3 để chọn các phần tử DOM và thao tác chúng theo lệnh. Điều này hoạt động trong mọi môi trường JavaScript:

```javascript
function drawChart(data) {
  if (!data || data.length === 0) return;

  const svg = d3.select("#chart"); // Chọn theo ID, class, hoặc phần tử DOM

  // Xóa nội dung trước đó
  svg.selectAll("*").remove();

  // Thiết lập kích thước
  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };

  // Tạo thang đo, trục và vẽ trực quan hóa
  // ... mã d3 ở đây ...
}

// Gọi khi dữ liệu thay đổi
drawChart(myData);
```

**Mẫu B: Kết xuất khai báo (cho các framework có tạo khuôn mẫu)**
Sử dụng d3 cho các tính toán dữ liệu (thang đo, bố cục) nhưng hiển thị các phần tử thông qua framework của bạn:

```javascript
function getChartElements(data) {
  const xScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value)])
    .range([0, 400]);

  return data.map((d, i) => ({
    x: 50,
    y: i * 30,
    width: xScale(d.value),
    height: 25,
  }));
}

// Trong React: {getChartElements(data).map((d, i) => <rect key={i} {...d} fill="steelblue" />)}
// Trong Vue: chỉ thị v-for trên mảng được trả về
// Trong vanilla JS: Tạo các phần tử thủ công từ dữ liệu được trả về
```

Sử dụng Mẫu A cho các trực quan hóa phức tạp với chuyển đổi, tương tác hoặc khi tận dụng toàn bộ khả năng của d3. Sử dụng Mẫu B cho các trực quan hóa đơn giản hơn hoặc khi framework của bạn ưa thích kết xuất khai báo.

### 3. Cấu trúc mã trực quan hóa

Tuân theo cấu trúc tiêu chuẩn này trong hàm vẽ của bạn:

```javascript
function drawVisualization(data) {
  if (!data || data.length === 0) return;

  const svg = d3.select("#chart"); // Hoặc truyền một bộ chọn/phần tử
  svg.selectAll("*").remove(); // Xóa lần kết xuất trước đó

  // 1. Xác định kích thước
  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  // 2. Tạo nhóm chính với lề
  const g = svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // 3. Tạo thang đo
  const xScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.x)])
    .range([0, innerWidth]);

  const yScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.y)])
    .range([innerHeight, 0]); // Lưu ý: đảo ngược cho tọa độ SVG

  // 4. Tạo và thêm các trục
  const xAxis = d3.axisBottom(xScale);
  const yAxis = d3.axisLeft(yScale);

  g.append("g").attr("transform", `translate(0,${innerHeight})`).call(xAxis);

  g.append("g").call(yAxis);

  // 5. Liên kết dữ liệu và tạo các phần tử trực quan
  g.selectAll("circle")
    .data(data)
    .join("circle")
    .attr("cx", (d) => xScale(d.x))
    .attr("cy", (d) => yScale(d.y))
    .attr("r", 5)
    .attr("fill", "steelblue");
}

// Gọi khi dữ liệu thay đổi
drawVisualization(myData);
```

### 4. Triển khai kích thước đáp ứng (Responsive Sizing)

Làm cho trực quan hóa phản hồi với kích thước vùng chứa:

```javascript
function setupResponsiveChart(containerId, data) {
  const container = document.getElementById(containerId);
  const svg = d3.select(`#${containerId}`).append("svg");

  function updateChart() {
    const { width, height } = container.getBoundingClientRect();
    svg.attr("width", width).attr("height", height);

    // Vẽ lại trực quan hóa với kích thước mới
    drawChart(data, svg, width, height);
  }

  // Cập nhật khi tải lần đầu
  updateChart();

  // Cập nhật khi thay đổi kích thước cửa sổ
  window.addEventListener("resize", updateChart);

  // Trả về hàm dọn dẹp
  return () => window.removeEventListener("resize", updateChart);
}

// Sử dụng:
// const cleanup = setupResponsiveChart('chart-container', myData);
// cleanup(); // Gọi khi component unmount hoặc phần tử bị xóa
```

Hoặc sử dụng ResizeObserver để theo dõi vùng chứa trực tiếp hơn:

```javascript
function setupResponsiveChartWithObserver(svgElement, data) {
  const observer = new ResizeObserver(() => {
    const { width, height } = svgElement.getBoundingClientRect();
    d3.select(svgElement).attr("width", width).attr("height", height);

    // Vẽ lại trực quan hóa
    drawChart(data, d3.select(svgElement), width, height);
  });

  observer.observe(svgElement.parentElement);
  return () => observer.disconnect();
}
```

## Các mẫu trực quan hóa phổ biến

### Biểu đồ cột (Bar chart)

```javascript
function drawBarChart(data, svgElement) {
  if (!data || data.length === 0) return;

  const svg = d3.select(svgElement);
  svg.selectAll("*").remove();

  const width = 800;
  const height = 400;
  const margin = { top: 20, right: 30, bottom: 40, left: 50 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  const g = svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  const xScale = d3
    .scaleBand()
    .domain(data.map((d) => d.category))
    .range([0, innerWidth])
    .padding(0.1);

  const yScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value)])
    .range([innerHeight, 0]);

  g.append("g")
    .attr("transform", `translate(0,${innerHeight})`)
    .call(d3.axisBottom(xScale));

  g.append("g").call(d3.axisLeft(yScale));

  g.selectAll("rect")
    .data(data)
    .join("rect")
    .attr("x", (d) => xScale(d.category))
    .attr("y", (d) => yScale(d.value))
    .attr("width", xScale.bandwidth())
    .attr("height", (d) => innerHeight - yScale(d.value))
    .attr("fill", "steelblue");
}

// Sử dụng:
// drawBarChart(myData, document.getElementById('chart'));
```

### Biểu đồ đường (Line chart)

```javascript
const line = d3
  .line()
  .x((d) => xScale(d.date))
  .y((d) => yScale(d.value))
  .curve(d3.curveMonotoneX); // Đường cong mượt mà

g.append("path")
  .datum(data)
  .attr("fill", "none")
  .attr("stroke", "steelblue")
  .attr("stroke-width", 2)
  .attr("d", line);
```

### Biểu đồ phân tán (Scatter plot)

```javascript
g.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", (d) => xScale(d.x))
  .attr("cy", (d) => yScale(d.y))
  .attr("r", (d) => sizeScale(d.size)) // Tùy chọn: mã hóa kích thước
  .attr("fill", (d) => colourScale(d.category)) // Tùy chọn: mã hóa màu sắc
  .attr("opacity", 0.7);
```

### Biểu đồ dây (Chord diagram)

Một biểu đồ dây hiển thị mối quan hệ giữa các thực thể trong bố cục hình tròn, với các dải ruy băng đại diện cho các luồng giữa chúng:

```javascript
function drawChordDiagram(data) {
  // định dạng dữ liệu: mảng các đối tượng với nguồn, đích và giá trị
  // Ví dụ: [{ source: 'A', target: 'B', value: 10 }, ...]

  if (!data || data.length === 0) return;

  const svg = d3.select("#chart");
  svg.selectAll("*").remove();

  const width = 600;
  const height = 600;
  const innerRadius = Math.min(width, height) * 0.3;
  const outerRadius = innerRadius + 30;

  // Tạo ma trận từ dữ liệu
  const nodes = Array.from(new Set(data.flatMap((d) => [d.source, d.target])));
  const matrix = Array.from({ length: nodes.length }, () =>
    Array(nodes.length).fill(0),
  );

  data.forEach((d) => {
    const i = nodes.indexOf(d.source);
    const j = nodes.indexOf(d.target);
    matrix[i][j] += d.value;
    matrix[j][i] += d.value;
  });

  // Tạo bố cục dây
  const chord = d3.chord().padAngle(0.05).sortSubgroups(d3.descending);

  const arc = d3.arc().innerRadius(innerRadius).outerRadius(outerRadius);

  const ribbon = d3
    .ribbon()
    .source((d) => d.source)
    .target((d) => d.target);

  const colourScale = d3.scaleOrdinal(d3.schemeCategory10).domain(nodes);

  const g = svg
    .append("g")
    .attr("transform", `translate(${width / 2},${height / 2})`);

  const chords = chord(matrix);

  // Vẽ các dải ruy băng
  g.append("g")
    .attr("fill-opacity", 0.67)
    .selectAll("path")
    .data(chords)
    .join("path")
    .attr("d", ribbon)
    .attr("fill", (d) => colourScale(nodes[d.source.index]))
    .attr("stroke", (d) => d3.rgb(colourScale(nodes[d.source.index])).darker());

  // Vẽ các nhóm (csung)
  const group = g.append("g").selectAll("g").data(chords.groups).join("g");

  group
    .append("path")
    .attr("d", arc)
    .attr("fill", (d) => colourScale(nodes[d.index]))
    .attr("stroke", (d) => d3.rgb(colourScale(nodes[d.index])).darker());

  // Thêm nhãn
  group
    .append("text")
    .each((d) => {
      d.angle = (d.startAngle + d.endAngle) / 2;
    })
    .attr("dy", "0.31em")
    .attr(
      "transform",
      (d) =>
        `rotate(${(d.angle * 180) / Math.PI - 90})translate(${outerRadius + 30})${d.angle > Math.PI ? "rotate(180)" : ""}`,
    )
    .attr("text-anchor", (d) => (d.angle > Math.PI ? "end" : null))
    .text((d, i) => nodes[i])
    .style("font-size", "12px");
}
```

### Bản đồ nhiệt (Heatmap)

Bản đồ nhiệt sử dụng màu sắc để mã hóa các giá trị trong lưới hai chiều, hữu ích để hiển thị các mẫu trên các danh mục:

```javascript
function drawHeatmap(data) {
  // định dạng dữ liệu: mảng các đối tượng với hàng, cột và giá trị
  // Ví dụ: [{ row: 'A', column: 'X', value: 10 }, ...]

  if (!data || data.length === 0) return;

  const svg = d3.select("#chart");
  svg.selectAll("*").remove();

  const width = 800;
  const height = 600;
  const margin = { top: 100, right: 30, bottom: 30, left: 100 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  // Lấy các hàng và cột duy nhất
  const rows = Array.from(new Set(data.map((d) => d.row)));
  const columns = Array.from(new Set(data.map((d) => d.column)));

  const g = svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // Tạo thang đo
  const xScale = d3
    .scaleBand()
    .domain(columns)
    .range([0, innerWidth])
    .padding(0.01);

  const yScale = d3
    .scaleBand()
    .domain(rows)
    .range([0, innerHeight])
    .padding(0.01);

  // Thang màu cho các giá trị
  const colourScale = d3
    .scaleSequential(d3.interpolateYlOrRd)
    .domain([0, d3.max(data, (d) => d.value)]);

  // Vẽ hình chữ nhật
  g.selectAll("rect")
    .data(data)
    .join("rect")
    .attr("x", (d) => xScale(d.column))
    .attr("y", (d) => yScale(d.row))
    .attr("width", xScale.bandwidth())
    .attr("height", yScale.bandwidth())
    .attr("fill", (d) => colourScale(d.value));

  // Thêm nhãn trục x
  svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .selectAll("text")
    .data(columns)
    .join("text")
    .attr("x", (d) => xScale(d) + xScale.bandwidth() / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .text((d) => d)
    .style("font-size", "12px");

  // Thêm nhãn trục y
  svg
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .selectAll("text")
    .data(rows)
    .join("text")
    .attr("x", -10)
    .attr("y", (d) => yScale(d) + yScale.bandwidth() / 2)
    .attr("dy", "0.35em")
    .attr("text-anchor", "end")
    .text((d) => d)
    .style("font-size", "12px");

  // Thêm chú thích màu
  const legendWidth = 20;
  const legendHeight = 200;
  const legend = svg
    .append("g")
    .attr("transform", `translate(${width - 60},${margin.top})`);

  const legendScale = d3
    .scaleLinear()
    .domain(colourScale.domain())
    .range([legendHeight, 0]);

  const legendAxis = d3.axisRight(legendScale).ticks(5);

  // Vẽ gradient màu trong chú thích
  for (let i = 0; i < legendHeight; i++) {
    legend
      .append("rect")
      .attr("y", i)
      .attr("width", legendWidth)
      .attr("height", 1)
      .attr("fill", colourScale(legendScale.invert(i)));
  }

  legend
    .append("g")
    .attr("transform", `translate(${legendWidth},0)`)
    .call(legendAxis);
}
```

### Biểu đồ tròn (Pie chart)

```javascript
const pie = d3
  .pie()
  .value((d) => d.value)
  .sort(null);

const arc = d3
  .arc()
  .innerRadius(0)
  .outerRadius(Math.min(width, height) / 2 - 20);

const colourScale = d3.scaleOrdinal(d3.schemeCategory10);

const g = svg
  .append("g")
  .attr("transform", `translate(${width / 2},${height / 2})`);

g.selectAll("path")
  .data(pie(data))
  .join("path")
  .attr("d", arc)
  .attr("fill", (d, i) => colourScale(i))
  .attr("stroke", "white")
  .attr("stroke-width", 2);
```

### Mạng hướng lực (Force-directed network)

```javascript
const simulation = d3
  .forceSimulation(nodes)
  .force(
    "link",
    d3
      .forceLink(links)
      .id((d) => d.id)
      .distance(100),
  )
  .force("charge", d3.forceManyBody().strength(-300))
  .force("center", d3.forceCenter(width / 2, height / 2));

const link = g
  .selectAll("line")
  .data(links)
  .join("line")
  .attr("stroke", "#999")
  .attr("stroke-width", 1);

const node = g
  .selectAll("circle")
  .data(nodes)
  .join("circle")
  .attr("r", 8)
  .attr("fill", "steelblue")
  .call(
    d3.drag().on("start", dragstarted).on("drag", dragged).on("end", dragended),
  );

simulation.on("tick", () => {
  link
    .attr("x1", (d) => d.source.x)
    .attr("y1", (d) => d.source.y)
    .attr("x2", (d) => d.target.x)
    .attr("y2", (d) => d.target.y);

  node.attr("cx", (d) => d.x).attr("cy", (d) => d.y);
});

function dragstarted(event) {
  if (!event.active) simulation.alphaTarget(0.3).restart();
  event.subject.fx = event.subject.x;
  event.subject.fy = event.subject.y;
}

function dragged(event) {
  event.subject.fx = event.x;
  event.subject.fy = event.y;
}

function dragended(event) {
  if (!event.active) simulation.alphaTarget(0);
  event.subject.fx = null;
  event.subject.fy = null;
}
```

## Thêm tương tác

### Chú giải công cụ (Tooltips)

```javascript
// Tạo div chú giải công cụ (bên ngoài SVG)
const tooltip = d3
  .select("body")
  .append("div")
  .attr("class", "tooltip")
  .style("position", "absolute")
  .style("visibility", "hidden")
  .style("background-color", "white")
  .style("border", "1px solid #ddd")
  .style("padding", "10px")
  .style("border-radius", "4px")
  .style("pointer-events", "none");

// Thêm vào các phần tử
circles
  .on("mouseover", function (event, d) {
    d3.select(this).attr("opacity", 1);
    tooltip
      .style("visibility", "visible")
      .html(`<strong>${d.label}</strong><br/>Value: ${d.value}`);
  })
  .on("mousemove", function (event) {
    tooltip
      .style("top", event.pageY - 10 + "px")
      .style("left", event.pageX + 10 + "px");
  })
  .on("mouseout", function () {
    d3.select(this).attr("opacity", 0.7);
    tooltip.style("visibility", "hidden");
  });
```

### Thu phóng và xoay (Zoom and pan)

```javascript
const zoom = d3
  .zoom()
  .scaleExtent([0.5, 10])
  .on("zoom", (event) => {
    g.attr("transform", event.transform);
  });

svg.call(zoom);
```

### Tương tác nhấp chuột

```javascript
circles.on("click", function (event, d) {
  // Xử lý nhấp chuột (gửi sự kiện, cập nhật trạng thái ứng dụng, v.v.)
  console.log("Clicked:", d);

  // Phản hồi trực quan
  d3.selectAll("circle").attr("fill", "steelblue");
  d3.select(this).attr("fill", "orange");

  // Tùy chọn: gửi sự kiện tùy chỉnh để framework/ứng dụng của bạn lắng nghe
  // window.dispatchEvent(new CustomEvent('chartClick', { detail: d }));
});
```

## Chuyển tiếp và hoạt ảnh

Thêm các chuyển tiếp mượt mà cho các thay đổi trực quan:

```javascript
// Chuyển tiếp cơ bản
circles.transition().duration(750).attr("r", 10);

// Chuyển tiếp chuỗi
circles
  .transition()
  .duration(500)
  .attr("fill", "orange")
  .transition()
  .duration(500)
  .attr("r", 15);

// Chuyển tiếp so le
circles
  .transition()
  .delay((d, i) => i * 50)
  .duration(500)
  .attr("cy", (d) => yScale(d.value));

// Easing tùy chỉnh
circles.transition().duration(1000).ease(d3.easeBounceOut).attr("r", 10);
```

## Tham khảo thang đo

### Thang đo định lượng

```javascript
// Thang đo tuyến tính
const xScale = d3.scaleLinear().domain([0, 100]).range([0, 500]);

// Thang đo log (cho dữ liệu hàm mũ)
const logScale = d3.scaleLog().domain([1, 1000]).range([0, 500]);

// Thang đo lũy thừa
const powScale = d3.scalePow().exponent(2).domain([0, 100]).range([0, 500]);

// Thang đo thời gian
const timeScale = d3
  .scaleTime()
  .domain([new Date(2020, 0, 1), new Date(2024, 0, 1)])
  .range([0, 500]);
```

### Thang đo thứ tự

```javascript
// Thang đo băng (cho biểu đồ cột)
const bandScale = d3
  .scaleBand()
  .domain(["A", "B", "C", "D"])
  .range([0, 400])
  .padding(0.1);

// Thang đo điểm (cho danh mục đường/phân tán)
const pointScale = d3.scalePoint().domain(["A", "B", "C", "D"]).range([0, 400]);

// Thang đo thứ tự (cho màu sắc)
const colourScale = d3.scaleOrdinal(d3.schemeCategory10);
```

### Thang đo tuần tự

```javascript
// Thang màu tuần tự
const colourScale = d3.scaleSequential(d3.interpolateBlues).domain([0, 100]);

// Thang màu phân kỳ
const divScale = d3.scaleDiverging(d3.interpolateRdBu).domain([-10, 0, 10]);
```

## Thực tiễn tốt nhất

### Chuẩn bị dữ liệu

Luôn xác thực và chuẩn bị dữ liệu trước khi trực quan hóa:

```javascript
// Lọc các giá trị không hợp lệ
const cleanData = data.filter((d) => d.value != null && !isNaN(d.value));

// Sắp xếp dữ liệu nếu thứ tự quan trọng
const sortedData = [...data].sort((a, b) => b.value - a.value);

// Phân tích cú pháp ngày tháng
const parsedData = data.map((d) => ({
  ...d,
  date: d3.timeParse("%Y-%m-%d")(d.date),
}));
```

### Tối ưu hóa hiệu suất

Đối với các tập dữ liệu lớn (>1000 phần tử):

```javascript
// Sử dụng canvas thay vì SVG cho nhiều phần tử
// Sử dụng quadtree để phát hiện va chạm
// Đơn giản hóa đường dẫn với d3.line().curve(d3.curveStep)
// Triển khai cuộn ảo cho danh sách lớn
// Sử dụng requestAnimationFrame cho hoạt ảnh tùy chỉnh
```

### Khả năng truy cập

Làm cho trực quan hóa có thể truy cập được:

```javascript
// Thêm nhãn ARIA
svg
  .attr("role", "img")
  .attr("aria-label", "Bar chart showing quarterly revenue");

// Thêm tiêu đề và mô tả
svg.append("title").text("Quarterly Revenue 2024");
svg
  .append("desc")
  .text("Bar chart showing revenue growth across four quarters");

// Đảm bảo độ tương phản màu đủ
// Cung cấp điều hướng bàn phím cho các yếu tố tương tác
// Bao gồm thay thế bảng dữ liệu
```

### Kiểu dáng

Sử dụng kiểu dáng nhất quán, chuyên nghiệp:

```javascript
// Xác định bảng màu trước
const colours = {
  primary: "#4A90E2",
  secondary: "#7B68EE",
  background: "#F5F7FA",
  text: "#333333",
  gridLines: "#E0E0E0",
};

// Áp dụng kiểu chữ nhất quán
svg
  .selectAll("text")
  .style("font-family", "Inter, sans-serif")
  .style("font-size", "12px");

// Sử dụng các đường lưới tinh tế
g.selectAll(".tick line")
  .attr("stroke", colours.gridLines)
  .attr("stroke-dasharray", "2,2");
```

## Các vấn đề phổ biến và giải pháp

**Vấn đề**: Các trục không xuất hiện

- Đảm bảo các thang đo có miền hợp lệ (kiểm tra các giá trị NaN)
- Xác minh trục được thêm vào đúng nhóm
- Kiểm tra bản dịch transform có đúng không

**Vấn đề**: Chuyển tiếp không hoạt động

- Gọi `.transition()` trước khi thay đổi thuộc tính
- Đảm bảo các phần tử có khóa duy nhất để liên kết dữ liệu thích hợp
- Kiểm tra xem các phụ thuộc useEffect có bao gồm tất cả dữ liệu thay đổi không

**Vấn đề**: Kích thước đáp ứng không hoạt động

- Sử dụng ResizeObserver hoặc trình lắng nghe thay đổi kích thước cửa sổ
- Cập nhật kích thước trong trạng thái để kích hoạt hiển thị lại
- Đảm bảo SVG có các thuộc tính width/height hoặc viewBox

**Vấn đề**: Vấn đề hiệu suất

- Giới hạn số lượng phần tử DOM (cân nhắc canvas cho >1000 mục)
- Debounce các trình xử lý thay đổi kích thước
- Sử dụng `.join()` thay vì các lựa chọn enter/update/exit riêng biệt
- Tránh hiển thị lại không cần thiết bằng cách kiểm tra các phụ thuộc

## Tài nguyên

### references/

Chứa các tài liệu tham khảo chi tiết:

- `d3-patterns.md` - Bộ sưu tập toàn diện các mẫu trực quan hóa và ví dụ mã
- `scale-reference.md` - Hướng dẫn đầy đủ về các thang đo d3 với các ví dụ
- `colour-schemes.md` - Các lược đồ màu D3 và đề xuất bảng màu

### assets/

Chứa các mẫu soạn sẵn:

- `chart-template.js` - Mẫu khởi đầu cho biểu đồ cơ bản
- `interactive-template.js` - Mẫu với chú giải công cụ, thu phóng và tương tác
- `sample-data.json` - Các tập dữ liệu ví dụ để kiểm thử

Các mẫu này hoạt động với vanilla JavaScript, React, Vue, Svelte hoặc bất kỳ môi trường JavaScript nào khác. Thích ứng chúng khi cần thiết cho framework cụ thể của bạn.

Để sử dụng các tài nguyên này, hãy đọc các tệp liên quan khi cần hướng dẫn chi tiết cho các loại trực quan hóa hoặc mẫu cụ thể.
