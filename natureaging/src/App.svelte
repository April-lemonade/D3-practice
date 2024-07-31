<script>
  import * as d3 from "d3";
  import Gradient from "../ components/Gradient.svelte";
  import Head from "../ components/Head.svelte";

  let width = window.innerWidth;
  let height = window.innerHeight;
  const dataUrl = "./data/data.csv";
  const { csv } = d3;
  let data = [];
  let xValue, yValue;
  let xScale, yScale;
  let xAxis, yAxis;
  const margin = 0;
  const text = [
    "Her husband lifeless in chair facing TV, whole days mute, her own mind, her hearing, shot. And it won’t get any better. Absolutely nothing to look forward to, she says to whom if not you?",
    "How she fights the sleeping pill, to stay up with me as I drive back from the beach, to keep talking, to speak and be spoken to: ",
    "the assurance of voices, even her own now druggy iterative slur which is like low birdsong, a blind inquiry in twilight—is someone there?",
    "—and a claim she is alive, here in the seat beside me, her seatbelt around her as she overcomes the pill, the devious obstacle course, ",
    "the drowsiness I administered in order to drive her four hours home this evening, to concentrate on driving, to save my pollen-swollen and mucus-",
    "inflamed throat from the overtime shift of talk, to stay the repetitive questions, her struggle among scraps and familiar names torn from faces and feelings, ",
    "the cipher-names of her husband and her grandchildren, a language of blanks, I drive with my left hand on the wheel, the other massaging the loose skin of her hands,",
    " feeling for the tightened cords in her palms, bands of fascia that curl her fingers forward, and when I think she’s finally fallen asleep,",
    " her face up-tilted and drawn, as we cross Indian River Inlet where she’d often, with my father, fished for flounder from the Grady-White whose single outboard,",
    " pumped and primed, they’d startled into coughs of blue smoke in the canal back in Bethany hours earlier and piloted through the gauntlet of buoys—red right return",
    "—that drew it through Assawoman Bay, past Harpoon Hanna’s, and into the yawning Atlantic—when I start to pull my hand away, she tightens her grip,",
    " not opening her eyes, maybe in reflex, holding on to her will to be awake with me, her son, whom she knows, whom she thanks now in her almost sleep, ",
    "her narcotic fatigue, in the spreading murk of the pill I coaxed her to swallow so that the trip might be easier while she rides beside me, holding on.",
  ];

  const main = async () => {
    data = await csv(dataUrl);
  };

  main();

  let groupedData = [];

  $: if (data.length > 0) {
    // data.reverse();
    // console.log(data);
    xValue = (d) => d.Protein;
    yValue = (d) => parseFloat(d.Level);

    xScale = d3
      .scalePoint()
      .domain(data.map(xValue))
      // .padding(0.2)
      .range([margin, width - margin]);

    yScale = d3
      .scaleLinear()
      .domain([d3.min(data, yValue), d3.max(data, yValue)])
      .range([height - margin, margin]);

    xAxis = d3.axisBottom(xScale);
    yAxis = d3.axisLeft(yScale);

    groupedData = Array.from(d3.group(data, (d) => d.target_yrs).entries());
  }
</script>

<!-- <div
  class="chart-container"
  role="contentinfo"
  
  style="height: 100vh;"
>
  <svg
    {height}
    {width}
    style="aspect-ratio: 3 / 4;width: auto;background: linear-gradient(to top, #213353 40%, #374e7a 55%, #60809f 65%, #99b3d0 75%,#f2f2f2 100%);"
    viewBox={`0 0 ${width} ${height}`}
  >
    {#each groupedData as [year, yearData], i}
      <path
        class="line"
        id={"curve" + i}
        fill="none"
        d={d3
          .line()
          .curve(d3.curveBasis)
          .x((d) => xScale(xValue(d)))
          .y((d) => yScale(yValue(d)) + (i - 6) * -100)(yearData)}
      />
      <text>
        <textPath
          xlink:href={"#curve" + i}
          style="font-size: 3rem;fill:white;font-family: Julee, cursive;"
          opacity={1 - i * 0.05}
        >
          {text[i]}{text[i]}
        </textPath>
      </text>
    {/each}
  </svg>
</div> -->
<div class="chart-container" style="height: 100vh;">
  <!-- <Gradient {data} {text} {xValue} {yValue} {margin}></Gradient> -->
  <Head {data} {xValue} {yValue} {margin}></Head>
</div>
