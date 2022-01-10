<script>
  import { createEventDispatcher } from "svelte";
  import cloud from "d3-cloud";
  import { select } from "d3-selection";
  import { scaleOrdinal } from "d3-scale";
  import * as CS from "d3-scale-chromatic";

  // event dispatcher
  const dispatch = createEventDispatcher();

  // color scheme
  const color_scheme = {
    schemeCategory10: CS.schemeCategory10,
    schemeAccent: CS.schemeAccent,
    schemeDark2: CS.schemeDark2,
    schemePaired: CS.schemePaired,
    schemePastel1: CS.schemePastel1,
    schemePastel2: CS.schemePastel2,
    schemeSet1: CS.schemeSet1,
    schemeSet2: CS.schemeSet2,
    schemeSet3: CS.schemeSet3,
    schemeTableau10: CS.schemeTableau10,
  };

  // props
  export let words = [];
  export let width = 800;
  export let height = 500;
  export let font = "Impact";
  export let maxFontSize = 50;
  export let minRotate = 0;
  export let maxRotate = 0;
  export let scheme = "schemeTableau10";
  export let padding = 10;
  export let backgroundColor = "#fff";

  // count max word occurrence
  const calculateMaxWordCount = (words) =>
    words.reduce((prev, cur) => (prev.count < cur.count ? prev : cur), {
      count: 1,
    }).count;
  let maxWordCount = calculateMaxWordCount(words);
  $: maxWordCount = calculateMaxWordCount(words);
  // text color scheme
  const fill = scaleOrdinal(color_scheme[scheme]);

  // events
  const onWordClick = (d) => dispatch("click", d);
  const onWordMouserOver = (d) => dispatch("mouseover", d);
  const onWordMouseOut = (d) => dispatch("mouseout", d);
  const onWordMouseMove = (d) => dispatch("mousemove", d);

  const wordCloudLayout = (node, words) => {
    const createLayout = (words) =>
      cloud()
        .size([width, height])
        .words(words)
        .padding(padding)
        .rotate(() => ~~(Math.random() * maxRotate) + minRotate)
        .font(font)
        .fontSize((d) => Math.floor((d.count / maxWordCount) * maxFontSize));
    const startLayout = (words) => {
      if (!words.length) {
        return;
      }
      const layout = createLayout(words);
      layout.on("end", draw(node, layout));
      layout.start();
    };
    const cleanLayout = () => {
      select(node).select("svg").remove();
    };
    startLayout(words);
    return {
      update: (newWords) => {
        cleanLayout();
        startLayout(newWords);
      },
      destroy: () => {
        cleanLayout();
      },
    };
  };

  function draw(target, layout) {
    return function (words) {
      select(target)
        .append("svg")
        .attr("width", layout.size()[0])
        .attr("height", layout.size()[1])
        .append("g")
        .attr(
          "transform",
          "translate(" + layout.size()[0] / 2 + "," + layout.size()[1] / 2 + ")"
        )
        .selectAll("text")
        .data(words)
        .enter()
        .append("text")
        .style("font-size", (d) => d.size + "px")
        .style("font-family", font)
        .style("fill", (_d, i) => fill(i))
        .attr("text-anchor", "middle")
        .attr(
          "transform",
          (d) => "translate(" + [d.x, d.y] + ")rotate(" + d.rotate + ")"
        )
        .text((d) => d.text)
        .on("click", onWordClick)
        .on("mouseover", onWordMouserOver)
        .on("mouseout", onWordMouseOut)
        .on("mousemove", onWordMouseMove);
    };
  }
</script>

<div
  class="wordcloud"
  use:wordCloudLayout={words}
  style="background-color: {backgroundColor};"
/>

<style>
  div.wordcloud {
    width: fit-content;
    height: fit-content;
  }
</style>
