<script>

  import {countries} from './countries';
  import {countries_no} from './countries_norsk';
  import {pages} from './pages';

    // const fetchData = (async () => {
	// 	// const response = await fetch('https://en.wikipedia.org/w/api.php?action=query&origin=*&prop=revisions&rvprop=content&rvsection=0&titles=storting&format=json', {method: "GET"})
	// 	const response = await fetch('https://en.wikipedia.org/w/api.php?&origin=*&action=parse&page=Storting&prop=text&format=json', {method: "GET"})
	// 	// const response = await fetch('https://en.wikipedia.org/w/api.php?&origin=*&action=parse&page=House_of_Representatives_(Netherlands)&prop=text&format=json', {method: "GET"})
  //   return await response.json();
	// })()

  async function fetchData(page){
    if(page == '' || !page){
      return '';
    }
		const response = await fetch(`https://en.wikipedia.org/w/api.php?&origin=*&action=parse&page=${page}&prop=text&format=json`, {method: "GET"})
    return await response.json();
  }

  function parseData(res){
    console.log(res);
    if(res == ''){
      return 'No data';
    }
    let parsed = JSON.stringify(res);
    parsed = parsed.substring(parsed.indexOf('Political groups'));
    parsed = parsed.substring(parsed.indexOf('<td'), parsed.indexOf('</td>'));
    console.log(parsed);
    // parsed = parsed.match(/(?<=(<li>)).+?(?=(<\/li>))/g);
    let colors = parsed.match(/(?<=<span)(.*?)(?=<\/span>)/g);
    colors = colors.join().match(/(?<=background-color:)(.*?)(?=;)/g);
    console.log(colors);
    
    let groups = parsed.match(/[a-zA-Z\s]+?(<\/a>)*\s\(\d*?\)(?!<\/b>)/gm);
    console.log(groups);

    if(parsed.indexOf('Government') > -1){
      
    }

    var result = [];
    for(let i = 0; i < groups.length; i++){
      groups[i] = groups[i].replace("</a>", "");
      colors[i] = colors[i].replace("background-color:", "");
      result.push({group: groups[i], color: colors[i]});
    }


    return result;
  }



	let name = 'World';
	let hoverColor = 'DodgerBlue';
	let defaultColor = 'grey';
	let hover = '';
	let vbx = 0;
	let vby = 0;
	let vbw = 2000;
	let vbh = 1000;
  let opx = 0;
  let opy = 0;
  let posX = 0;
  let posY = 0;
	let selected = '';
	let selectedColor = 'RoyalBlue'
	
	function handleMouseOver(e) {
		hover = e.target.id;
	}
	
	function handleMouseOut(e){
		hover = '';
	}
	
	function handleClick(e){
		selected = e.target.id;
	}
	
	function doNothing(){
		
	}
	
	function zoomIn(){
		vbw -= 250;
		vbh -= 125;
	}
	
	function zoomOut(){
		vbw += 250;
		vbh += 125;

    if(vbw == 2000){
      vbx = 0;
      vby = 0;
      opx = 0;
      opy = 0;
    }
	}

  function onMouseMove(e){
    vbx = opx - (e.offsetX - posX)*vbw/2000;
    vby = opy - (e.offsetY - posY)*vbh/1000;
  }

  function onMouseDown(e){
    addEventListener('mousemove', onMouseMove);
    addEventListener('mouseup', onMouseUp);
    posX = e.offsetX;
    posY = e.offsetY;
  }

  function onMouseUp(){
    removeEventListener('mousemove', onMouseMove);
    removeEventListener('mouseup', onMouseUp);
    vbx > 2000 ? vbx = 2000 : '';
    vby > 1000? vby = 1000 : '';
    vbx < -1000 ? vbx = -1000 : '';
    vby < -500 ? vby = -500 : '';
    opx = vbx;
    opy = vby;
  }

  function onMouseWheel(e){

      var w = vbw;
      var h = vbh;
      var mx = e.offsetX;  
      var my = e.offsetY;    
      var dw = w*Math.sign(e.deltaY)*-0.1;
      var dh = h*Math.sign(e.deltaY)*-0.1;
      var dx = dw*mx/2000;
      var dy = dh*my/1000;

      if(vbw - dw > 2000){
        vbx = 0;
        vby = 0;
        opx = 0;
        opy = 0;
        vbw = 2000;
        vbh = 1000;
      }
      else if(vbw - dw < 2){
        vbw = 2;
        vbh = 1;
      }
      else{
        vbw -= dw;
        vbh -= dh;

        vbx += dx;
        vby += dy;

        vbx > 2000 ? vbx = 2000 : '';
        vby > 1000? vby = 1000 : '';
        vbx < -1000 ? vbx = -1000 : '';
        vby < -500 ? vby = -500 : '';

        opx = vbx;
        opy = vby;
      }
  }

  function resetMap(){
    vbx = 0;
    vby = 0;
    opx = 0;
    opy = 0;
    vbw = 2000;
    vbh = 1000;
    selected = '';
  }
	
</script>

<style>
	svg path {
		outline: none;
	}

  .map-button{
    font-size: x-large;
    border: darkgray 2px solid;
    fill: whitesmoke;
    user-select: none;
  }
  
/* 
  .zoomButton {
    position: absolute;
    width: 50px;
    height: 50px;
    padding: none;
    font-weight: 900;
    font-size: xx-large;
    line-height: 0;
  }

  .zoomButton:nth-of-type(1) {
    top: 120px;
  }

  .zoomButton:nth-of-type(2) {
    top: 180px;
  } */

</style>

<body style="height: 100%; overflow: hidden;">

  <div style="position: absolute; top: 100px;" hidden={selected == ''}>
    {#await fetchData(pages[selected])}
    <p>...waiting</p>
  {:then data}
    {#if data != ''}
      {#each parseData(data) as group}
      <p> <span style="padding-right: 20px; margin-right: 20px; background-color: {group.color}; border: solid darkgrey 1px;"></span> {group.group} </p>
      {/each}
    {:else}
      <p> No Data </p>
    {/if}

    
  {:catch error}
    <p>{error}</p>
  {/await}
  </div>

  <div style="width: 100%; position: absolute; background-color: rgb(255,255,255,0.8); display: flex; justify-content: space-around">
      <button class="map-button" on:click={resetMap}>Reset</button>

      <!-- <h4>
        {vbx.toFixed(0)} {vby.toFixed(0)} {vbw.toFixed(0)} {vbh.toFixed(0)}
      </h4>

      <h4>hover: {hover ? countries[hover] : ''}</h4>
      
      <h4>selected: {selected ? countries[selected] : ''}</h4> -->

      <h1> selected: {selected ? countries[selected] + ' | ' + countries_no[selected]: ''}</h1>
  </div>
  
  <!-- <button class="zoomButton" on:click={zoomIn} disabled={vbw <= 250}>+</button>
  <button class="zoomButton" on:click={zoomOut} disabled={vbw >= 2000}>-</button> -->
  
  
  <div>
    <svg id="map" xmlns="http://www.w3.org/2000/svg" viewBox="{vbx+' '+vby+' '+vbw+' '+vbh}" on:mousedown={onMouseDown} on:mousewheel={onMouseWheel}>
  
        <!-- copy map path in here -->
  
        <path id="AF" fill={selected == 'AF' ? selectedColor :hover == 'AF' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1369.9,333.8h-5.4l-3.8-0.5l-2.5,2.9l-2.1,0.7l-1.5,1.3l-2.6-2.1l-1-5.4l-1.6-0.3v-2l-3.2-1.5l-1.7,2.3l0.2,2.6
        l-0.6,0.9l-3.2-0.1l-0.9,3l-2.1-1.3l-3.3,2.1l-1.8-0.8l-4.3-1.4h-2.9l-1.6-0.2l-2.9-1.7l-0.3,2.3l-4.1,1.2l0.1,5.2l-2.5,2l-4,0.9
        l-0.4,3l-3.9,0.8l-5.9-2.4l-0.5,8l-0.5,4.7l2.5,0.9l-1.6,3.5l2.7,5.1l1.1,4l4.3,1.1l1.1,4l-3.9,5.8l9.6,3.2l5.3-0.9l3.3,0.8l0.9-1.4
        l3.8,0.5l6.6-2.6l-0.8-5.4l2.3-3.6h4l0.2-1.7l4-0.9l2.1,0.6l1.7-1.8l-1.1-3.8l1.5-3.8l3-1.6l-3-4.2l5.1,0.2l0.9-2.3l-0.8-2.5l2-2.7
        l-1.4-3.2l-1.9-2.8l2.4-2.8l5.3-1.3l5.8-0.8l2.4-1.2l2.8-0.7L1369.9,333.8L1369.9,333.8z"/>
      <path id="AO" fill={selected == 'AO' ? selectedColor : hover == 'AO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1068.3,609.6l-16.6-0.1l-1.9,0.7l-1.7-0.1l-2.3,0.9l-0.5,1.2l2.8,4l1.1,4.3l1.6,6.1l-1.7,2.6l-0.3,1.3l1.3,3.8
        l1.5,3.9l1.6,2.2l0.3,3.6l-0.7,4.8l-1.8,2.8l-3.3,4.2l-1.3,2.6l-1.9,5.7l-0.3,2.7l-2,5.9l-0.9,5.5l0.5,4l2.7-1.2l3.3-1l3.6,0.1
        l3.2,2.9l0.9-0.4l22.5-0.3l3.7,3l13.4,0.9l10.3-2.5l-3.5-4l-3.6-5.2l0.8-20.3l11.6,0.1l-0.5-2.2l0.9-2.4l-0.9-3l0.7-3l-0.5-2
        l-2.6-0.4l-3.5,1l-2.4-0.2l-1.4,0.6l0.5-7.6l-1.9-2.3l-0.3-4l0.9-3.8l-1.2-2.4v-4h-6.8l0.5-2.3h-2.9l-0.3,1.1l-3.4,0.3l-1.5,3.7
        l-0.9,1.6l-3-0.9l-1.9,0.9l-3.7,0.5l-2.1-3.3l-1.3-2.1l-1.6-3.8L1068.3,609.6L1068.3,609.6z M1046.5,608.3l0.2-2.7l0.9-1.7l2-1.3
        l-2-2.2l-1.8,1.1l-2.2,2.7l1.4,4.8L1046.5,608.3L1046.5,608.3z"/>
      <path id="AL" fill={selected == 'AL' ? selectedColor : hover == 'AL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1077.5,300.5l-2,3.1l0.5,1.9l0,0l1,1l-0.5,1.9l-0.1,4.3l0.7,3l3,2.1l0.2,1.4l1,0.4l2.1-3l0.1-2.1l1.6-0.9V312
        l-2.3-1.6l-0.9-2.6l0.4-2.1l0,0l-0.5-2.3l-1.3-0.6l-1.3-1.6l-1.3,0.5L1077.5,300.5L1077.5,300.5z"/>
      <path id="AE" fill={selected == 'AE' ? selectedColor : hover == 'AE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1283.9,408.6l-1.3-2.2l-3,3.9l-3.7,4.1l-3.3,4.3l-3.3-0.2l-4.6-0.2l-4.2,1l-0.3-1.7l-1,0.3l0.4,1.5l2.6,6.4
        l16.8,3.2l1-1.3l-0.1-2.6l1.4-2.6l-0.3-2.6l2.4-1.3l-1.1-0.8l0.1-4.2h2.8L1283.9,408.6L1283.9,408.6z"/>
      <path id="AR" fill={selected == 'AR' ? selectedColor : hover == 'AR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M669.8,920.7l0.9-3l-7.3-1.5l-7.7-3.6l-4.3-4.6l-3-2.8l5.9,13.5h5l2.9,0.2l3.3,2.1L669.8,920.7L669.8,920.7z
        M619.4,712.6l-7.4-1.5l-4,5.7l0.9,1.6l-1.1,6.6l-5.6,3.2l1.6,10.6l-0.9,2l2,2.5l-3.2,4l-2.6,5.9l-0.9,5.8l1.7,6.2l-2.1,6.5
        l4.9,10.9l1.6,1.2l1.3,5.9l-1.6,6.2l1.4,5.4l-2.9,4.3l1.5,5.9l3.3,6.3l-2.5,2.4l0.3,5.7l0.7,6.4l3.3,7.6l-1.6,1.2l3.6,7.1l3.1,2.3
        l-0.8,2.6l2.8,1.3l1.3,2.3l-1.8,1.1l1.8,3.7l1.1,8.2l-0.7,5.3l1.8,3.2l-0.1,3.9l-2.7,2.7l3.1,6.6l2.6,2.2l3.1-0.4l1.8,4.6l3.5,3.6
        l12,0.8l4.8,0.9l2.2,0.4l-4.7-3.6l-4.1-6.3l0.9-2.9l3.5-2.5l0.5-7.2l4.7-3.5l-0.2-5.6l-5.2-1.3l-6.4-4.5l-0.1-4.7l2.9-3.1l4.7-0.1
        l0.2-3.3l-1.2-6.1l2.9-3.9l4.1-1.9l-2.5-3.2l-2.2,2l-4-1.9l-2.5-6.2l1.5-1.6l5.6,2.3l5-0.9l2.5-2.2l-1.8-3.1l-0.1-4.8l-2-3.8
        l5.8,0.6l10.2-1.3l6.9-3.4l3.3-8.3l-0.3-3.2l-3.9-2.8l-0.1-4.5l-7.8-5.5l-0.3-3.3l-0.4-4.2l0.9-1.4l-1.1-6.3l0.3-6.5l0.5-5.1
        l5.9-8.6l5.3-6.2l3.3-2.6l4.2-3.5l-0.5-5.1l-3.1-3.7l-2.6,1.2l-0.3,5.7l-4.3,4.8l-4.2,1.1l-6.2-1l-5.7-1.8l4.2-9.6l-1.1-2.8
        l-5.9-2.5l-7.2-4.7l-4.6-1L632,713.7l-1-1.3l-6.3-0.3l-1.6,5.1L619.4,712.6L619.4,712.6z"/>
      <path id="AM" fill={selected == 'AM' ? selectedColor : hover == 'AM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1219,325.1l-0.9-4.4l-2.5-1.1l-2.5-1.7l1-2l-3.1-2.2l0.7-1.5l-2.2-1.1l-1.4-1.7l-6.9,1l1.3,2.2v3.1l4.2,1.5
        l2.4,1.9l1-0.2l1.8,1.7h2.3l0.2,1l2.8,3.7L1219,325.1L1219,325.1z"/>
      <path id="AU" fill={selected == 'AU' ? selectedColor : hover == 'AU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1726.7,832l-3-0.5l-1.9,2.9l-0.6,5.4l-2.1,4l-0.5,5.3l3,0.2l0.8,0.3l6.6-4.3l0.6,1.7l4-4.9l3.2-2.2l4.5-7.3
        l-2.8-0.5l-4.8,1.2l-3.4,0.9L1726.7,832L1726.7,832z M1776.8,659.7l0.5-2.3l0.1-3.6l-1.6-3.2l0.1-2.7l-1.3-0.8l0.1-3.9l-1.2-3.2
        l-2.3,2.4l-0.4,1.8l-1.5,3.5l-1.8,3.4l0.6,2.1l-1.2,1.3l-1.5,4.8l0.1,3.7l-0.7,1.8l0.3,3.1l-2.6,5l-1.3,3.5l-1.7,2.9l-1.7,3.4
        l-4.1,2.1l-4.9-2.1l-0.5-2l-2.5-1.6h-1.6l-3.3-3.8l-2.5-2.2l-3.9-2l-3.9-3.5l-0.1-1.8l2.5-3.1l2.1-3.2l-0.3-2.6l1.9-0.2l2.5-2.5
        l2-3.4l-2.2-3.2l-1.5,1.2l-2-0.5l-3.5,1.8l-3.2-2l-1.7,0.7l-4.5-1.6l-2.7-2.7l-3.5-1.5l-3.1,0.9l3.9,2.1l-0.3,3.2l-4.8,1.2l-2.8-0.7
        l-3.6,2.2l-2.9,3.7l0.6,1.5l-2.7,1.7l-3.4,5.1l0.6,3.5l-3.4-0.6h-3.5l-2.5-3.8l-3.7-2.9l-2.8,0.8l-2.6,0.9l-0.3,1.6l-2.4-0.7
        l-0.3,1.8l-3,1.1l-1.7,2.5l-3.5,3.1l-1.4,4.8l-2.3-1.3l-2.2,3.1l1.5,3l-2.6,1.2l-1.4-5.5l-4.8,5.4l-0.8,3.5l-0.7,2.5l-3.8,3.3
        l-2,3.4l-3.5,2.8l-6.1,1.9l-3.1-0.2l-1.5,0.6l-1.1,1.4l-3.5,0.7l-4.7,2.4l-1.4-0.8l-2.6,0.5l-4.6,2.3l-3.2,2.7l-4.8,2.1l-3.1,4.4
        l0.4-4.8l-3.1,4.6l-0.1,3.7l-1.3,3.2l-1.5,1.5l-1.3,3.7l0.9,1.9l0.1,2l1.6,5l-0.7,3.3l-1-2.5l-2.3-1.8l0.4,5.9l-1.7-2.8l0.1,2.8
        l1.8,5l-0.6,5l1.7,2.5l-0.4,1.9l0.9,4.1l-1.3,3.6l-0.3,3.6l0.7,6.5l-0.7,3.7l-2.2,4.4l-0.6,2.3l-1.5,1.5l-2.9,0.8l-1.5,3.7l2.4,1.2
        l4,4.1h3.6l3.8,0.3l3.3-2.1l3.4-1.8l1.4,0.3l4.5-3.4l3.8-0.3l4.1-0.7l4.2,1.2l3.6-0.6l4.6-0.2l3-2.6l2.3-3.3l5.2-1.5l6.9-3.2l5,0.4
        l6.9-2.1l7.8-2.3l9.8-0.6l4,3.1l3.7,0.2l5.3,3.8l-1.6,1.5l1.8,2.4l1.3,4.6l-1.6,3.4l2.9,2.6l4.3-5.1l4.3-2.1l6.7-5.5l-1.6,4.7
        l-3.4,3.2l-2.5,3.7l-4.4,3.5l5.2-1.2l4.7-4.4l-0.9,4.8l-3.2,3.1l4.7,0.8l1.3,2.6l-0.4,3.3l-1.5,4.9l1.4,4l4,1.9l2.8,0.4l2.4,1
        l3.5,1.8l7.2-4.7l3.5-1.2l-2.7,3.4l2.6,1.1l2.7,2.8l4.7-2.7l3.8-2.5l6.3-2.7l6-0.2l4.2-2.3l0.9-2l3-4.5l3.9-4.8l3.6-3.2l4.4-5.6
        l3.3-3.1l4.4-5l5.4-3.1l5-5.8l3.1-4.5l1.4-3.6l3.8-5.7l2.1-2.9l2.5-5.7l-0.7-5.4l1.7-3.9l1.1-3.7v-5.1l-2.8-5.1l-1.9-2.5l-2.9-3.9
        l0.7-6.7l-1.5,1l-1.6-2.8l-2.5,1.4l-0.6-6.9l-2.2-4l1-1.5l-3.1-2.8l-3.2-3l-5.3-3.3l-0.9-4.3l1.3-3.3l-0.4-5.5l-1.3-0.7l-0.2-3.2
        l-0.2-5.5l1.1-2.8l-2.3-2.5l-1.4-2.7l-3.9,2.4L1776.8,659.7L1776.8,659.7z"/>
      <path id="AT" fill={selected == 'AT' ? selectedColor : hover == 'AT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1060.2,264l-2.3-1.2l-2.3,0.3l-4-1.9l-1.7,0.5l-2.6,2.5l-3.8-2l-1.5,2.9l-1.7,0.8l1,4l-0.4,1.1l-1.7-1.3l-2.4-0.2
        l-3.4,1.2l-4.4-0.3l-0.6,1.6l-2.6-1.7l-1.5,0.3l0.2,1.1l-0.7,1.6l2.3,1.1l2.6,0.2l3.1,0.9l0.5-1.2l4.8-1.1l1.3,2.2l7.2,1.6l4.2,0.4
        l2.4-1.4l4.3-0.1l0.9-1.1l1.3-4l-1.1-1.3h2.8l0.2-2.6l-0.7-2.1L1060.2,264L1060.2,264z"/>
      <path id="AZ" fill={selected == 'AZ' ? selectedColor : hover == 'AZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1210.1,318.9l-1,0.2l1.2,2.4l3.2,2.9l3.7,0.9l-2.8-3.7l-0.2-1h-2.3L1210.1,318.9L1210.1,318.9z M1220.5,309.6
        l-4.3-3.8l-1.5-0.2l-1.1,0.9l3.2,3.4l-0.6,0.7l-2.8-0.4l-4.2-1.8l-1.1,1l1.4,1.7l2.2,1.1l-0.7,1.5l3.1,2.2l-1,2l2.5,1.7l2.5,1.1
        l0.9,4.4l5.3-4.7l1.9-0.5l1.9,1.9l-1.2,3.1l3.8,3.4l1.3-0.3l-0.8-3.2l1.7-1.5l0.4-2.2l-0.1-5l4.2-0.5l-2-1.7l-2.5-0.2l-3.5-4.5
        l-3.4-3.2l0,0l-2.6,2.5l-0.5,1.5L1220.5,309.6L1220.5,309.6z"/>
      <path id="BI" fill={selected == 'BI' ? selectedColor : hover == 'BI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1148.2,590l-0.3-2.5l0,0l-3-0.4l-1.7,3.6l-3.5-0.5l1.4,2.9l0.1,1.1l2,6.1l-0.1,0.3l0.6-0.1l2.1-2.3l2.2-3.3
        l1.4-1.4v-2L1148.2,590L1148.2,590z"/>
      <path id="BE" fill={selected == 'BE' ? selectedColor : hover == 'BE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1000.7,246.2l-4.4,1.3l-3.6-0.5l0,0l-3.8,1.2l0.7,2.2l2.2,0.1l2.4,2.4l3.4,2.9l2.5-0.4l4.4,2.8l0.4-3.5l1.3-0.2
        l0.4-4.2l-2.8-1.4L1000.7,246.2L1000.7,246.2z"/>
      <path id="BJ" fill={selected == 'BJ' ? selectedColor : hover == 'BJ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M996.9,498l-4.3-3.7h-2l-1.9,1.9l-1.2,1.9l-2.7,0.6l-1.2,2.8l-1.9,0.7l-0.7,3.3l1.7,1.9l2,2.3l0.2,3.1l1.1,1.3
        l-0.2,14.6l1.4,4.4l4.6-0.8l0.3-10.2L992,518l1-4l1.7-1.9l2.7-4l-0.6-1.7l1.1-2.5l-1.2-3.8L996.9,498L996.9,498z"/>
      <path id="BF" fill={selected == 'BF' ? selectedColor : hover == 'BF' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M978.8,477.2h-3.6l-1.4-1.2l-3,0.9l-5.2,2.6l-1.1,2l-4.3,2.9l-0.8,1.6l-2.3,1.3l-2.7-0.9l-1.6,1.6l-0.8,4.4
        l-4.5,5.2l0.2,2.2l-1.6,2.7l0.4,3.7l2.5,1.4l1,2.1l2.5,1.3l1.9-1.6l2.7-0.2l3.8,1.6l-0.8-4.8l0.2-3.6l9.7-0.3l2.4,0.5l1.8-1l2.6,0.5
        l4.9,0.1l1.9-0.7l1.2-2.8l2.7-0.6l1.2-1.9l0.1-4.4l-6.4-1.4l-0.2-3.1l-3.1-4.1l-0.8-2.9L978.8,477.2L978.8,477.2z"/>
      <path id="BD" fill={selected == 'BD' ? selectedColor : hover == 'BD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1486.5,431.9l-4.5-10.1l-1.5,0.1l-0.2,4l-3.5-3.3l1.1-3.6l2.4-0.4l1.6-5.3l-3.4-1.1l-5,0.1l-5.4-0.9l-1.2-4.4
        l-2.7-0.4l-4.8-2.7l-1.2,4.3l4.6,3.4l-3.1,2.4l-0.8,2.3l3.7,1.7l-0.4,3.8l2.6,4.8l1.6,5.2l2.2,0.6l1.7,0.7l0.6-1.2l2.5,1.3l1.3-3.5
        l-0.9-2.6l5.1,0.2l2.8,3.7l1.5,3.1l0.8,3.2l2,3.3l-1.1-5.1l2.1,1L1486.5,431.9L1486.5,431.9z"/>
      <path id="BG" fill={selected == 'BG' ? selectedColor : hover == 'BG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1121.6,294.3l-3-0.7l-4-2.2l-5.8,1.4l-2.3,1.6l-7.5-0.3l-4-1l-1.9,0.5l-1.8-2.6l-1.1,1.4l0.7,2.3l2.8,2.6l-1.7,1.9
        l-0.7,2l0.6,0.7l-0.7,0.9l2.8,2l0.8,4.1l3.8,0.2l3.9-1.7l3.9,2.1l4.6-0.6l-0.3-3l5-2l4.5,0.8l-2.1-3.5l1.3-4.4L1121.6,294.3
        L1121.6,294.3z"/>
      <path id="BA" fill={selected == 'BA' ? selectedColor : hover == 'BA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1062.2,284.9l-2.3,0.1l-1,1.3l-1.9-1.4l-0.9,2.5l2.7,2.9l1.3,1.9l2.5,2.3l2,1.4l2.2,2.5l4.7,2.4l0.4-3.4l1.5-1.4
        l0.9-0.6l1.2-0.3l0.5-2.9l-2.7-2.3l1-2.7h-1.8l0,0l-2.4-1.4l-3.5,0.1L1062.2,284.9L1062.2,284.9z"/>
      <path id="BY" fill={selected == 'BY' ? selectedColor : hover == 'BY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1112.8,219.4l-5.2-1.5l-4.6,2.3l-2.6,1l0.9,2.6l-3.5,2l-0.5,3.4l-4.8,2.2h-4.6l0.6,2.7l1.7,2.3l0.3,2.4l-2.7,1.2
        l1.9,2.9l0.5,2.7l2.2-0.3l2.4-1.6l3.7-0.2l5,0.5l5.6,1.5l3.8,0.1l2,0.9l1.6-1.1l1.5,1.5l4.3-0.3l2,0.6l-0.2-3.1l1.2-1.4l4.1-0.3l0,0
        l-2-3.9l-1.5-2l0.8-0.6l3.9,0.2l1.6-1.3l-1.7-1.6l-3.4-1.1l0.1-1.1l-2.2-1.1l-3.7-3.9l0.6-1.6l-1-2.9l-4.8-1.4l-2.3,0.7
        L1112.8,219.4L1112.8,219.4z"/>
      <path id="BZ" fill={selected == 'BZ' ? selectedColor : hover == 'BZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M482.5,471.1l1.4-2.2l1-0.2l1.3-1.7l1-3.2l-0.3-0.6l0.9-2.3l-0.4-1l1.3-2.7l0.3-1.8h-1.1l0.1-0.9h-1l-2.5,3.9
        l-0.9-0.8l-0.7,0.3l-0.1,1l-0.7,5l-1.2,7.2L482.5,471.1L482.5,471.1z"/>
      <path id="BO" fill={selected == 'BO' ? selectedColor : hover == 'BO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M655.7,700.5l1.6-1.3l-0.8-3.6l1.3-2.8l0.5-5l-1.6-4l-3.2-1.7l-0.8-2.6l0.6-3.6l-10.7-0.3l-2.7-7.4l1.6-0.1
        l-0.3-2.8l-1.2-1.8l-0.5-3.7l-3.3-1.9l-3.5,0.1l-2.5-1.9l-3.8-1.2l-2.4-2.4l-6.3-1l-6.4-5.7l0.3-4.3l-0.9-2.5l0.4-4.7l-7.3,1.1
        l-2.8,2.3l-4.8,2.6l-1.1,1.9l-2.9,0.2l-4.2-0.6l5.5,10.3l-1.1,2.1l0.1,4.5l0.3,5.4l-1.9,3.2l1.2,2.4l-1.1,2.1l2.8,5.3L591,684
        l3.1,4.3l1.2,4.6l3.2,2.7l-1.1,6.2l3.7,7.1l3.1,8.8l3.8-0.9l4-5.7l7.4,1.5l3.7,4.6l1.6-5.1l6.3,0.3l1,1.3l1.5-7.6l-0.2-3.4l2.1-5.6
        l9.5-1.9l5.1,0.1l5.4,3.3L655.7,700.5L655.7,700.5z"/>
      <path id="BR" fill={selected == 'BR' ? selectedColor : hover == 'BR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M659,560.1l-1.4,0.2l-3.1-0.5l-1.8,1.7l-2.6,1.1l-1.7,0.2l-0.7,1.3l-2.7-0.3l-3.5-3l-0.3-2.9l-1.4-3.3l1-5.4
        l1.6-2.2l-1.2-3l-1.9-0.9l0.8-2.8l-1.3-1.5l-2.9,0.3l0.7,1.8l-2.1,2.4l-6.4,2.4l-4,1l-1.7,1.5l-4.4-1.6l-4.2-0.8l-1,0.6l2.4,1.6
        l-0.3,4.3l0.7,4l4.8,0.5l0.3,1.4l-4.1,1.8l-0.7,2.7l-2.3,1l-4.2,1.5l-1.1,1.9l-4.4,0.5l-3-3.4l-1.1,0.8l-1-3.8l-1.6-2l-1.9,2.2
        l-10.9-0.1v3.9l3.3,0.7l-0.2,2.4l-1.1-0.6l-3.2,1v4.6l2.5,2.4l0.9,3.6l-0.1,2.8l-2.2,17.4l-5.1-0.3l-0.7,1l-4.6,1.2l-6.2,4.3l-0.4,3
        l-1.3,2.2l0.7,3.4l-3.3,1.9l0.1,2.7L562,620l2.6,5.8l3.3,3.8l-1,2.8l3.7,0.3l2.3,3.4l4.9,0.2l4.4-3.8l0.2,9.7l2.6,0.7l3-1.1l4.2,0.6
        l2.9-0.2l1.1-1.9l4.8-2.6l2.8-2.3l7.3-1.1l-0.4,4.7l0.9,2.5l-0.3,4.3l6.4,5.7l6.3,1l2.4,2.4l3.8,1.2l2.5,1.9l3.5-0.1l3.3,1.9
        l0.5,3.7l1.2,1.8l0.3,2.8l-1.6,0.1l2.7,7.4l10.7,0.3l-0.6,3.6l0.8,2.6l3.2,1.7l1.6,4l-0.5,5l-1.3,2.8l0.8,3.6l-1.6,1.3l1.9,3.6
        l0.4,8.6l6,1.2l2.1-1.2l3.9,1.7l1.2,1.9l1,5.8l0.9,2.5l2,0.3l2-1.1l2.1,1.2l0.3,3.5l-0.3,3.8l-0.7,3.6l2.6-1.2l3.1,3.7l0.5,5.1
        l-4.2,3.5l-3.3,2.6l-5.3,6.2l-5.9,8.6l3.4-0.7l6.2,4.9l1.9-0.2l6.2,4.1l4.8,3.5l3.8,4.3l-1.9,3l2.1,3.7l2.9-3.7l1.5-6l3.2-3l3.9-5
        l4.5-11.2l3.4-3.5l0.8-3.1l0.3-6.4l-1.3-3.5l0.3-4.8l4.1-6.3l6-5.1l6-1.8l3.6-2.9l8.5-2.4h5.9l1.1-3.8l4.2-2.8l0.6-6.5l5.1-8.3
        l0.5-8.5l1.6-2.6l0.3-4.1l1.1-9.9l-1-11.9l1.4-4.7l1.4-0.1l3.9-5.5l3.3-7.2l7.7-8.8l2.7-4.2l2-10.5l-1-3.9l-2-8.1l-2.1-2l-4.8-0.2
        l-4.3-1.9l-7.3-7.1l-8.4-5.3l-8.4,0.3l-10.9-3.4l-6.5,2l0.8-3.5l-2.7-3.8l-9.4-3.8l-7.1-2.3l-4.2,4.1l-0.3-6.3l-9.9-1l-1.7-2
        l4.2-5.2l-0.1-4.4l-3-1l-3-11.2l-1.3-3.5l-1.9,0.3l-3.5,5.8l-1.8,4.7l-2.1,2.4l-2.7,0.5l-0.8-1.8l-1.2-0.3l-1.8,1.8l-2.4-1.3
        l-3.2-1.4l-2.7,0.7l-2.3-0.6l-0.5,1.8l0.9,1.3l-0.5,1.3L659,560.1L659,560.1z"/>
      <path id="BN" fill={selected == 'BN' ? selectedColor : hover == 'BN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1617.8,543.4l2.7,3.3l1.1-2.2l2.7,0.2l0.1-4.1l0.1-3.1l-4.6,3.5L1617.8,543.4L1617.8,543.4z"/>
      <path id="BT" fill={selected == 'BT' ? selectedColor : hover == 'BT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1474.7,395.5l-2.7-1.8l-2.9-0.1l-4.2-1.5l-2.6,1.6l-2.6,4.8l0.3,1.2l5.5,2.5l3.2-1l4.7,0.4l4.4-0.2l-0.4-3.9
        L1474.7,395.5L1474.7,395.5z"/>
      <path id="BW" fill={selected == 'BW' ? selectedColor : hover == 'BW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1116.7,685l-1-0.5l-3.2,1.5h-1.6l-3.7,2.5l-2-2.6l-8.6,2.2l-4.1,0.2l-0.9,22.7l-5.4,0.2l-0.6,18.5l1.4,1l3,6.1
        l-0.7,3.8l1.1,2.3l4-0.7l2.8-2.8l2.7-1.9l1.5-3.1l2.7-1.5l2.3,0.8l2.5,1.8l4.4,0.3l3.6-1.5l0.6-2l1.2-3l3-0.5l1.7-2.4l2-4.3l5.2-4.7
        l8-4.7l-3.4-2.9l-4.2-0.9l-1.5-4.1l0.1-2.2l-2.3-0.7l-6-7l-1.6-3.7l-1.1-1.1L1116.7,685L1116.7,685z"/>
      <path id="CF" fill={selected == 'CF' ? selectedColor : hover == 'CF' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1110.5,517.3l-0.5-0.3l-2-1.8l-0.3-2l0.8-2.6V508l-3.3-4l-0.7-2.7l-3.5,1.1l-2.8,2.5l-4,7l-5.2,2.9l-5.4-0.4
        l-1.6,0.6l0.6,2.3l-2.9,2.2l-2.3,2.5l-7.1,2.4l-1.4-1.4l-0.9-0.2l-1,1.7l-4.7,0.4l-2.7,6.5l-1.4,1.1l-0.4,5l0.6,2.7l-0.4,1.9
        l2.6,3.3l0.5,2.3l2.1,3.2l2.6,2.1l0.3,2.9l0.6,1.8l2.9-5.9l3.3-3.4l3.8,1.1l3.6,0.4l0.5-4.5l2.2-3.2l3-2l4.6,2.1l3.6,2.4l4.1,0.6
        l4.2,1.2l1.6-3.8l0.8-0.5l2.6,0.6l6.2-3.1l2.2,1.3l1.8-0.2l0.9-1.5l2-0.6l4.3,0.7l3.6,0.1l1.8-0.6l-0.9-2.1l-4.2-2.5l-1.5-3.8
        l-2.4-2.7l-3.8-3.4l-0.1-2l-3.1-2.6L1110.5,517.3L1110.5,517.3z"/>
      <path id="CA" fill={selected == 'CA' ? selectedColor : hover == 'CA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M659,276.7l-0.7-3l-2.5,1.9l0.5,2.1l5.6,2.6l1.9-0.4l3.3-2.5l-4.7,0.1L659,276.7L659,276.7z M673.4,260.8l0.2-1.1
        l-4.1-2.6l-5.9-1.6l-1.9,0.6l3.5,2.9l5.7,1.9L673.4,260.8L673.4,260.8z M368.1,264.5l0.2-3.4l-3.2-2.6l-0.4-2.9l-0.1-2.1l-4.1-0.7
        l-2.4-0.9l-4.1-1.4l-1.4,1.5l-0.6,3.3l4.3,1.1l-0.4,1.8l2.9,2.2v2.2l6.3,2.8L368.1,264.5L368.1,264.5z M704.2,251l3.9-3.8l1.4-1.7
        l-2.1-0.3l-4.9,2.2l-4.2,3.5l-8.1,9.8l-5.3,3.7l1.6,1.7l-3.8,2.2l0.2,1.9l9.6,0.1l5.4-0.3l4.4,1.5l-4.4,2.9l2.9,0.2l7.3-5.4l1.2,0.8
        l-2.5,5.1l3,1.2l2.3-0.2l3.5-5.5l-0.5-3.9l0.3-3.3l-3.7,1.1l2.8-4.6l-4.3-1.9l-2.7,1.5l-3.9-1.7l2.4-2.1l-2.9-1.3l-3.8,2L704.2,251
        L704.2,251z M347.4,229.8l-1.9,2l-1.4,2.6l0.9,1.9l-0.6,2.8l0.7,2.8h1.9l-0.2-4.9l7.1-6.9l-4.9,0.5L347.4,229.8L347.4,229.8z
        M628.3,182.8l-0.4-1.2l-1.7-0.1l-2.8,1.7l-0.4,0.4l0.1,1.7l1.7,0.5L628.3,182.8L628.3,182.8z M618.7,179.6l0.8-1.1l-6-0.1l-4.9,2.7
        v1.5l3,0.2L618.7,179.6L618.7,179.6z M615.6,163l-2.7-0.5l-5,5.2l-3.6,4.4l-5.7,2.8l6.3-0.6l-0.8,3.4l8.2-3l6.2-3l0.8,2.6l5.9,1.3
        l4.9-1.8l-1.9-1.8l-3.4,0.4l1.3-2.7l-3.7-1.7l-3.4-1.9l-1.5-1.5l-2.8,0.9L615.6,163L615.6,163z M660.2,154.8l3.7-1.7l1-0.7l1.4-2.3
        l-2.3-1.5l-4.2,0.7l-3.8,3.1l-0.7,2.6L660.2,154.8L660.2,154.8z M586.4,144.1l-0.8-2l-0.3-1l-1.6-1l-3-1.5l-4.9,2.3l-5,1.7l3.5,2.4
        l3.8-0.6l4.1,1.6L586.4,144.1z M608.8,142l-6.6-1l5.7-2.6l-0.4-6l-1.9-2.3l-4.5-0.8l-8.1,3.8l-5.5,5.8l2.9,2.1l1.6,3.3l-6.3,5.5
        l-3.2-0.2l-6.2,4.4l4.2-5.2l-4.8-1.8l-4.5,0.9l-2.4,3.4l-5.9-0.1l-7.2,0.8l-5.1-2.4l-5,0.4l-1.5-2.9l-2.1-1.3l-3.8,0.5l-5.2,0.3
        l-4.4,1.8l2,2.3l-7,2.8l-1.4-3.3l-4.4,1l-11.8,0.6l-6.4-1.2l8.5-2.6l-2.8-2.8l-4.4,0.4l-4.7-1l-7.5-1.9l-3.8-2.3l-4.5-0.3l-3.3,1.6
        l-5.9,0.9l3.9-4.1l-9.4,3.6l-1.4-4.7l-2.1-0.6l-3.8,2.5l-4.5,1.2l-0.2-2.2l-8.2,1.4l-8.8,2.3l-5.2-0.6l-7,1.6l-6.2,2.3l-3.7-0.5
        l-3.3-2.6l-5.9-1.3l0,0l-24.3,20.2l-35.4,32.4l4.2,0.1l2.7,1.6l0.6,2.6l0.2,3.9l7.6-3.3l6.4-1.9l-0.5,3l0.7,2.4l1.7,2.7l-1.1,4.2
        l-1.5,6.8l4.6,3.8l-3.1,3.7l-5.1,2.9l0,0l-2.5,3.1l2.1,4.4l-3.1,4.9l4.1,2.6l-3.6,3.7l-1.3,5.5l6.9,2.5l1.6,2.7l5.4,6.1h0.7h13.9
        h14.6h4.8h15h14.5h14.7h14.8h16.7h16.8h10.1l1.3-2.4h1.6l-0.8,3.4l1,1l3.2,0.4l4.6,1l3.8,1.9l4.4-0.8l5.3,1.6l0,0l3.2-2.4l3.2-1
        l1.8-1.5l1.5-0.8l4,1.2l3.3,0.2l0.8,0.8l0.1,3.5l5.2,1l-1.7,1.7l1.2,1.9l-1.9,2.3l1.8,0.8l-1.9,2.1l0,0l1.2,0.2l1.3-0.9l0.5,1.4
        l3.4,0.7l3.8,0.1l3.8,0.6l4,1.2l0.8,2l1.4,4.7l-2.4,2l-3.8-0.8l-1-3.8l-0.9,3.9l-3.8,3.4l-0.8,2.9l-1.1,1.7l-4.1,2l0,0l-3.7,3.4
        l-2,2.2l2.7,0.4l4.5-2l2.9-1.7l1.6-0.3l2.6,0.6l1.7-0.9l2.8-0.8l4.7-0.8l0,0l0,0l0.3-1.8l-0.3,0.1l-1.7,0.3l-1.8-0.6l2.3-2.1
        l1.9-0.7l3.9-0.9l4.6-0.9l1.8,1.2l1.9-1.4l1.9-0.8l0.9,0.4l0.1,0.1l6.7-4.2l2.7-1.2h7.7h9.3l1-1.6l1.7-0.3l2.5-0.9l2.7-2.8l3.2-4.9
        l5.5-4.7l1.1,1.7l3.7-1.1l1.5,1.8l-2.8,8.5l2.1,3.5l5.9-0.8l8.1-0.2l-10.4,5.1l-1.5,5.2l3.7,0.5l7.1-4.5l5.8-2.4l12.2-3.7l7.5-4.1
        l-2.6-2.2l1-4.5l-7.1,7l-8.6,0.8l-5.5-3.1l-0.1-4.6l0.6-6.8l6.1-4.1l-3.3-3.1l-7.6,0.6l-12.1,5.2l-10.9,8.2l-4.6,1l7.8-5.7l10.1-8.3
        l7.2-2.7l5.7-4.4l5.2-0.5l7.3,0.1l10,1.3l8.6-1l7.8-5.1l8.7-2.2l4.2-2.1l4.2-2.3l2-6.8l-1.1-2.3l-3.4-0.8v-5.1l-2.3-1.9l-6.9-1.6
        l-2.8-3.4l-4.8-3.4l3.4-3.7l-2-7.1l-2.6-7.5l-1-5.2l-4.3,2.7l-7.4,6.5l-8.1,3.2l-1.6-3.4l-3.7-1l2.2-7.3l2.6-4.9l-7.7-0.5l-0.1-2.2
        l-3.6-3.3l-3-2l-4.5,1.5l-4.2-0.5l-6.6-1.6l-3.9,1.3l-3.8,9l-1,5.3l-8.8,6.1l3.1,4.5l0.5,5l-1.7,4l-4.7,4.1L610,224l-9,2.8l1.7,3.2
        l-2.2,9.6l-5.6,6.3l-4.6,1.9l-4.4-5.8l-0.1-6.8l1.7-6l3.6-5.2l-4.8-0.6l-7.5-0.4l-3.6-2.5l-4.8-1.6l-1.7-2.9l-3.3-2.2l-7-2.6
        l-7.1,1.2l0.7-4.5l1.5-5.5l-6-1l4.9-6.8l4.9-4.6l9.4-6.5l8.6-4.6l5.6-0.7l2.9-3.7l5.1-2.4l6.4-0.4l7.7-3.8l2.9-2.4l7.4-4.7l3.2-2.8
        l3.2,1.7l6.5-0.9L637,155l2.3-2.7l-0.8-2.9l5-2.9l1.7-2.7l-3.5-2.6l-5.4-0.8l-5.5-0.4l-4.6,5.9l-6.5,4.6l-7.2,4l-1.3-3.7l4.2-4
        l-2.2-3.5l-8.7,4.2L608.8,142L608.8,142z M533.3,123.1l-2.8-1l-14.1,3.2l-5.1,2l-7.8,3.9l5.4,1.4l6.2-0.1l-11.5,2.1v1.9l5.6,0.1
        l9-0.4l6.5,1.2l-6.2,1l-5.5-0.3l-7.1,0.9l-3.3,0.6l0.6,4.2l4.2-0.6l4.1,1.5l-0.3,2.5l7.8-0.5l11.2-0.8l9.4-1.8l5-0.4l5.7,1.5
        l6.7,0.8l3.1-1.9l-0.7-2.1l7-0.4l2.6-2.4l-5-2.5l-4.2-2.6l2.4-3.6l2.7-5.1l-2.2-2l-3-0.9l-4.2,0.8l-2.8,5.3l-4.3,2.1l2.2-5.1
        l-1.7-1.7l-7.3,2.7L539,124l-10.4,1.5L533.3,123.1L533.3,123.1z M572.4,121.6l-1.7-1.1l-5.4,0.2l-2.1,0.7l2.2,3.6
        C565.4,125,572.4,121.6,572.4,121.6z M680.1,123.2l-4.4-2.8l-8.4-0.5l-2.1,0.3l-1.7,1.8l2,2.8l0.9,0.3l4.8-0.7l4.1,0.1l4.1,0.1
        L680.1,123.2L680.1,123.2z M640.7,122.9l5.7-3.2l-11.2,1.3l-5.8,2.1l-7.1,4.6l-3.3,5.2l5.6,0.1l-6.1,2.3l1.8,1.9l5.9,0.8l7.3,1.5
        l13.8,1.2l7.9-0.6l3.2-1.6l2,1.8l3.3,0.3l2,3.3l-3.5,1.4l7.1,1.8l4.6,2.6l0.5,1.9L674,154l-8.6,5.4l-3.2,2.7l0.2,2l-9.2,0.7l-8,0.1
        l-5.4,4.2l2.4,1.9l13-0.9l0.9-1.6l4.7,2.7l4.7,2.9l-2.4,1.6l3.8,2.8l7.6,3.3l10.7,2.3l0.3-2l-2.8-3.5l-3.5-4.9l8.5,4.6l4.7,1.5
        l3.6-4.1v-5.6l-1-1.5l-4.4-2.5l-2.7-3.3l2.3-3.2l5.8-0.7l3.8,5.4l4,2.4l10.7-6.5l3.3-3.9l-6.4-0.3l-3.2-5.1l-5.9-1.2l-7.7-3.5l9-2.5
        l-0.8-5l-2.2-2.1l-8.3-2.1l-1.9-3.3l-8.2,1.2l1.1-2.3l-3.6-2.5l-6.8-2.6l-5.2,2.1l-9,1.5l3.3-3.4l-2.3-5.3l-11.6,2.1l-7.1,4.1
        L640.7,122.9L640.7,122.9z M590.7,119.5l-7.1,2.4l0.9,3.4l-7.4-0.7l-1.7,1.7l5.8,3.9l0.9,2l3.4,0.5l8.4-2l5.1-4.7l-3.8-2.2l6-2.4
        l0.5-1.5l-7.5,0.6L590.7,119.5L590.7,119.5z M613,124.9l5.6-1l10-4.5l-6.1-1.2l-7.8-0.2l-5.2,1.4l-4.2,2.1l-2.5,2.6l-1.8,4.5
        l4.3,0.2L613,124.9z M498.3,132.1l2.6-2.3l9.1-3.6l13.8-3.6l6.4-1.3l-1.6-2.1l-1.9-1.5l-9.4-0.2l-4.1-1.1l-14,0.8l-0.3,3.1l-7.6,3.3
        l-7.4,3.8l-4.3,2.2l5.9,2.7l-0.6,2.3L498.3,132.1L498.3,132.1z M622.4,113.8l0.3-1.6l-1.4-1.7l-6.9,1.3L610,114l3.2,1.3l5.1,0.4
        L622.4,113.8L622.4,113.8z M613.7,105.2l-1.1,0.7l-4.8-0.3l-7.6,1.6l-3.8-0.1l-4.3,3.8l6.6-0.4l-3.4,2.9l3.2,0.8l6.8-0.5l5.8-3.7
        l2.8-2.5L613.7,105.2z M574.6,107.7l1.8-2.3l-3.1-0.5l-5.7,1.7l-0.7,4.7l-6.1-0.4L558,108l-8.2-1.6l-5.4,1.4l-11.6,4.8l4.1,0.8
        l17.8-0.5l-10.6,2.2l-1.5,1.6l5.9-0.1l12.2-2.2l13.8-0.8l5.1-2.3l2.3-2.4l-3.7-0.2l-4.3,0.8C573.9,109.5,574.6,107.7,574.6,107.7z
        M629.8,103.4l-7.1-0.3l-3.8,2l2.6,1.5l7,0.6l1.4,2.1l-2.2,2.4l-1.5,2.8l8.5,1.6l5.5,0.6l8-0.1l11.6-0.8l4.3,0.6l6.7-1l3.5-1.4l1-2
        l-2.3-1.9l-5.8-0.3l-8,0.4l-7,1.1l-5.1-0.4l-4.8-0.3l-1.2-1.1l-3.1-1.1l2.8-1.9l-1.4-1.6l-7.3,0.1L629.8,103.4L629.8,103.4z
        M554.8,100.8l-6,0.7l-5.5-0.1l-12.1,3.1l-11.6,3.7l0,0l3.6,1l7-0.7l9.8-2.1l3.8-0.3l5.2-1.6L554.8,100.8z M635.3,101.4l1-0.5
        l-1.5-0.9l-7.2-0.1l-0.6,1.3l6.4,0.3L635.3,101.4L635.3,101.4z M576.9,100.6l3.2-1.4l-4.1-0.8l-5.9,0.5l-5.1,1.5l3.3,1.5
        C568.3,101.9,576.9,100.6,576.9,100.6z M584.7,96.4l-3.3-0.9l-1.6-0.2l-5.7,1.3l-1,0.7h6L584.7,96.4z M631.1,98.9l3-1.7l-2.3-1.6
        l-1.7-0.3l-4.4-0.1l-2.1,1.8l-0.7,1.8l1.6,1.1L631.1,98.9L631.1,98.9z M617.4,97.7l0.1-2.2l-7.4-1.7l-6.1-0.6l-2.1,1.7l2.8,1.1
        l-5.3,1.4l7.7,0.2l4,1.5l5.2,0.5L617.4,97.7z M671.1,91.6l0.6-2.8L667,88l-4.7-0.9l-1.6-2.2l-8.2,0.2l0.3,0.9l-3.9,0.3l-4.1,1.3
        l-4.9,1.9l-0.3,1.9l2,1.5h6.5l-4.3,1.2l-2.1,1.6l1.6,1.9l6.7,0.6l6.8-0.4l10.5-3.4l6.4-1.3L671.1,91.6z M749.6,77.8l-7-0.2l-6.9-0.3
        l-10.2,0.6l-1.4-0.4l-10.3,0.2l-6.4,0.4l-5.1,0.6l-5,2l-2.3-1l-3.9-0.2l-6.7,1.4l-7.4,0.6l-4.1,0.1l-6,0.8l-1.1,1.3l2.5,1.2l0.8,1.6
        l4.4,1.5l12.4-0.3l7.2,0.5l-7.2,1.5l-2.2-0.4l-9.3-0.2l-1.1,2.2l3,1.7l-2.8,1.6l-7.5,1.1l-4.9,1.7l4.8,0.9l1.7,3l-7.5-2l-2.5,0.3
        l-2,3.4l-8,1.1l-2,2.3l6.7,0.3l4.9,0.6l11.7-0.8l8.4,1.4l12.6-3l1-1.1l-6.4,0.2l0.5-1.1l6.5-1.4l3.6-1.9l6.8-1.3l5-1.6l-0.8-2.2
        l3.3-0.8l-4.3-0.6l11.1-0.4l3.2-0.9l7.9-0.8l9.3-3.5l6.8-1.1l10.3-2.5h-7.4l3.9-0.9l9-0.8l9.7-1.6l1.1-1.1l-5.2-1l-6.7-0.4
        L749.6,77.8L749.6,77.8z"/>
      <path id="CH" fill={selected == 'CH' ? selectedColor : hover == 'CH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1024.3,270.6l-5.4-1.9l-1,1.4h-4.2l-1.3,1l-2.3-0.6l0.2,1.6l-3.5,3.5v2.8l2.4-0.9l1.8,2.7l2.2,1.3l2.4-0.3l2.7-2.1
        l0.9,1l2.4-0.2l0.9-2.5l3.8,0.8l2.1-1.1l0.3-2.5l-2.6-0.2l-2.3-1.1l0.7-1.6L1024.3,270.6L1024.3,270.6z"/>
      <path id="CL" fill={selected == 'CL' ? selectedColor : hover == 'CL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M648.4,905.2l-3.7-0.7l-3.3,2.5l0.2,4.1l-1.2,2.8l-7.2-2.2l-8.6-4l-4.5-1.3l9.7,6.8l6.3,3.2l7.5,3.4l5.3,0.9
        l4.3,1.8l3,0.5l2.3,0.1l3.2-1.8l0.5-2.4l-2.9-0.2h-5L648.4,905.2L648.4,905.2z M601.1,708.9l-3.7-7.1l1.1-6.2l-3.2-2.7l-1.2-4.6
        L591,684l-1.2,3.3l-2.7,1.6l2.1,9l1.5,10.4l-0.1,14.2v13.2l0.9,12.3l-1.9,7.8l2.1,7.8l-0.5,5.3l3.2,9.5l-0.1,9.5l-1.2,10.2
        l-0.6,10.5l-2.1,0.2l2.4,7.3l3.3,6.3l-1.1,4.3l1.9,11.6l1.5,8.8l3.5,0.9l-1.1-7.7l4,1.6l1.8,12.7l-6.4-2.1l2,10.2l-2.7,5.5l8.2,1.8
        l-3.4,4.8l0.2,6l5,10.6l4.2,4.1l0.2,3.6l3.3,3.8l7.5,3.5l0,0l7.4,4.2l6.2,2l2-0.1l-1.8-5.7l3.4-2.2l1.7-1.5h4.2l-4.8-0.9l-12-0.8
        l-3.5-3.6l-1.8-4.6l-3.1,0.4l-2.6-2.2l-3.1-6.6l2.7-2.7l0.1-3.9l-1.8-3.2l0.7-5.3l-1.1-8.2l-1.8-3.7l1.8-1.1l-1.3-2.3l-2.8-1.3
        l0.8-2.6l-3.1-2.3l-3.6-7.1l1.6-1.2l-3.3-7.6l-0.7-6.4l-0.3-5.7l2.5-2.4l-3.3-6.3l-1.5-5.9l2.9-4.3l-1.4-5.4l1.6-6.2l-1.3-5.9
        l-1.6-1.2l-4.9-10.9l2.1-6.5l-1.7-6.2l0.9-5.8l2.6-5.9l3.2-4l-2-2.5l0.9-2l-1.6-10.6l5.6-3.2l1.1-6.6l-0.9-1.6l-3.8,0.9L601.1,708.9
        L601.1,708.9z"/>
      <path id="CN" fill={selected == 'CN' ? selectedColor : hover == 'CN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1587.2,453.3l0.6-3.6l2-2.8l-1.6-2.5l-3.2-0.1l-5.8,1.8l-2.2,2.8l1,5.5l4.9,2L1587.2,453.3L1587.2,453.3z
        M1600.4,256.8l-6.1-6.1l-4.4-3.7l-3.8-2.7l-7.7-6.1l-5.9-2.3l-8.5-1.8l-6.2,0.2l-5.1,1.1l-1.7,3l3.7,1.5l2.5,3.3l-1.2,2l0.1,6.5
        l1.9,2.7l-4.4,3.9l-7.3-2.3l0.6,4.6l0.3,6.2l2.7,2.6l2.4-0.8l5.4,1l2.5-2.3l5.1,2l7.2,4.3l0.7,2.2l-4.3-0.7l-6.8,0.8l-2.4,1.8
        l-1.4,4.1l-6.3,2.4l-3.1,3.3l-5.9-1.3l-3.2-0.5l-0.4,4l2.9,2.3l1.9,2.1l-2.5,2l-1.9,3.3l-4.9,2.2l-7.5,0.2l-7.2,2.2l-4.4,3.3l-3.2-2
        l-6.2,0.1l-9.3-3.8l-5.5-0.9l-6.4,0.8l-11.2-1.3l-5.5,0.1l-4.7-3.6l-4.9-5.7l-3.4-0.7l-7.9-3.8l-7.2-0.9l-6.4-1l-3-2.7l-1.3-7.3
        l-5.8-5l-8.1-2.3l-5.7-3.3l-3.3-4.4l-1.7,0.5l-1.8,4.2l-3.8,0.6l2.5,6.2l-1.6,2.8l-10.7-2l1,11.1l-2,1.4l-9,2.4l8.7,10.7l-2.9,1.6
        l1.7,3.5l-0.2,1.4l-6.8,3.4l-1,2.4l-6.4,0.8l-0.6,4l-5.7-0.9l-3.2,1.2l-4,3l1.1,1.5l-1,1.5l3,5.9l1.6-0.6l3.5,1.4l0.6,2.5l1.8,3.7
        l1.4,1.9l4.7,3l2.9,5l9.4,2.6l7.6,7.5l0.8,5.2l3,3.3l0.6,3.3l-4.1-0.9l3.2,7l6.2,4l8.5,4.4l1.9-1.5l4.7,2l6.4,4.1l3.2,0.9l2.5,3.1
        l4.5,1.2l5,2.8l6.4,1.5l6.5,0.6l3-1.4l1.5,5.1l2.6-4.8l2.6-1.6l4.2,1.5l2.9,0.1l2.7,1.8l4.2-0.8l3.9-4.8l5.3-4l4.9,1.5l3.2-2.6
        l3.5,3.9l-1.2,2.7l6.1,0.9l3-0.4l2.7,3.7l2.7,1.5l1.3,4.9l0.8,5.3l-4.1,5.3l0.7,7.5l5.6-1l2.3,5.8l3.7,1.3l-0.8,5.2l4.5,2.4l2.5,1.2
        l3.8-1.8l0.6,2.6l0.7,1.5l2.9,0.1l-1.9-7.2l2.7-1l2.7-1.5h4.3l5.3-0.7l4.1-3.4l3,2.4l5.2,1.1l-0.2,3.7l3,2.6l5.9,1.6l2.4-1l7.7,2
        l-0.9,2.5l2.2,4.6l3-0.4l0.8-6.7l5.6-0.9l7.2-3.2l2.5-3.2l2.3,2.1l2.8-2.9l6.1-0.7l6.6-5.3l6.3-5.9l3.3-7.6l2.3-8.4l2.1-6.9l2.8-0.5
        l-0.1-5.1l-0.8-5.1l-3.8-2l-2.5-3.4l2.8-1.7l-1.6-4.7l-5.4-4.9l-5.4-5.8l-4.6-6.3l-7.1-3.5l0.9-4.6l3.8-3.2l1-3.5l6.7-1.8l-2.4-3.4
        l-3.4-0.2l-5.8-2.5l-3.9,4.6l-4.9-1.9l-1.5-2.9l-4.7-1l-4.7-4.4l1.2-3l5-0.3l1.2-4.1l3.6-4.4l3.4-2.2l4.4,3.3l-1.9,4.2l2.3,2.5
        l-1.4,3l4.8-1.8l2.4-2.9l6.3-1.9l2.1-4l3.8-3.4l1-4.4l3.6,2l4.6,0.2l-2.7-3.3l6.3-2.6l-0.1-3.5l5.5,3.6l0,0l-1.9-3.1l2.5-0.1
        l-3.8-7.3l-4.7-5.3l2.9-2.2l6.8,1.1l-0.6-6l-2.8-6.8l0.4-2.3l-1.3-5.6l-6.9,1.8l-2.6,2.5h-7.5l-6-5.8l-8.9-4.5L1600.4,256.8
        L1600.4,256.8z"/>
      <path id="CI" fill={selected == 'CI' ? selectedColor : hover == 'CI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M946.5,506.2l-2.3,0.9l-1.3,0.8l-0.9-2.7l-1.6,0.7l-1-0.1l-1,1.9l-4.3-0.1l-1.6-1l-0.7,0.6l-1.1,0.5l-0.5,2.2
        l1.3,2.6l1.3,5.1l-2,0.8l-0.6,0.9l0.4,1.2l-0.3,2.8h-0.9l-0.3,1.8l0.6,3.1l-1.2,2.8l1.6,1.8l1.8,0.4l2.3,2.7l0.2,2.5l-0.5,0.8
        l-0.5,5.2l1.1,0.2l5.6-2.4l3.9-1.8l6.6-1.1l3.6-0.1l3.9,1.3l2.6-0.1l0.2-2.5l-2.4-5.5l1.5-7.2l2.3-5.3l-1.4-9.1l-3.8-1.6l-2.7,0.2
        l-1.9,1.6l-2.5-1.3l-1-2.1L946.5,506.2L946.5,506.2z"/>
      <path id="CM" fill={selected == 'CM' ? selectedColor : hover == 'CM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1060.1,502.9l0.2-4.3l-0.5-4.2l-2.2-4.1l-1.6,0.4l-0.2,2l2.3,2.6l-0.6,1.1l-0.3,2.1l-4.6,5l-1.5,4l-0.7,3.3
        l-1.2,1.4l-1.1,4.5l-3,2.6l-0.8,3.2l-1.2,2.6l-0.5,2.6l-3.9,2.2l-3.2-2.6l-2.1,0.1l-3.3,3.7l-1.6,0.1l-2.7,6.1l-1.4,4.5v1.8l1.4,0.9
        l1.1,2.8l2.6,1.1l2.2,4.2l-0.8,5l9.2,0.2l2.6-0.4l3.4,0.8l3.4-0.8l0.7,0.3l7.1,0.3l4.5,1.7l4.5,1.5l0.4-3.5l-0.6-1.8l-0.3-2.9
        l-2.6-2.1l-2.1-3.2l-0.5-2.3l-2.6-3.3l0.4-1.9l-0.6-2.7l0.4-5l1.4-1.1l2.7-6.5l0.9-1.7l-1.8-4.4l-0.8-2.6l-2.5-1.1l-3.3-3.7l1.2-3
        l2.5,0.6l1.6-0.4l3.1,0.1L1060.1,502.9L1060.1,502.9z"/>
      <path id="CD" fill={selected == 'CD' ? selectedColor : hover == 'CD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1124.9,539.4l-4.3-0.7l-2,0.6l-0.9,1.5l-1.8,0.2l-2.2-1.3l-6.2,3.1l-2.6-0.6l-0.8,0.5l-1.6,3.8l-4.2-1.2l-4.1-0.6
        l-3.6-2.4l-4.6-2.1l-3,2l-2.2,3.2l-0.5,4.5l-0.3,3.8l-1.6,3.4l-1.1,4l-0.7,5.6l0.3,3.6l-0.9,2.2l-0.2,2.4l-0.6,2l-3.7,3.1l-2.6,3.2
        l-2.5,6.2l0.2,5.3l-1.4,2l-3.3,3.1l-3.4,4l-2-1.1l-0.4-1.8l-3.1-0.1l-1.9,2.4l-1.5-0.6l-2,1.3l-0.9,1.7l-0.2,2.7l-1.5,0.7l0.8,2
        l2.3-0.9l1.7,0.1l1.9-0.7l16.6,0.1l1.3,4.7l1.6,3.8l1.3,2.1l2.1,3.3l3.7-0.5l1.9-0.9l3,0.9l0.9-1.6l1.5-3.7l3.4-0.3l0.3-1.1h2.9
        l-0.5,2.3h6.8v4l1.2,2.4l-0.9,3.8l0.3,4l1.9,2.3l-0.5,7.6l1.4-0.6l2.4,0.2l3.5-1l2.6,0.4l1.9,0.1l0.3,2l2.6-0.1l3.5,0.6l1.8,2.8
        l4.5,0.9l3.4-2l1.2,3.4l4.3,0.8l2,2.8l2.1,3.5h4.3l-0.3-6.9l-1.5,1.2l-3.9-2.5l-1.4-1.1l0.8-6.4l1.2-7.5l-1.2-2.8l1.6-4.1l1.6-0.7
        l7.5-1.1l1,0.3l0.2-1.1l-1.5-1.7l-0.7-3.5l-3.4-3.5l-1.8-4.5l1-2.7l-1.5-3.6l1.1-10.2l0.1,0.1l-0.1-1.1l-1.4-2.9l0.6-3.5l0.8-0.4
        l0.2-3.8l1.6-1.8l0.1-4.8l1.3-2.4l0.3-5.1l1.2-3l2.1-3.3l2.2-1.7l1.8-2.3l-2.3-0.8l0.3-7.5l0,0l-5-4.2l-1.4-2.7l-3.1,1.3l-2.6-0.4
        l-1.5,1.1l-2.5-0.8l-3.5-5.2l-1.8,0.6L1124.9,539.4L1124.9,539.4z"/>
      <path id="CG" fill={selected == 'CG' ? selectedColor : hover == 'CG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1080.3,549.9l-3.6-0.4l-3.8-1.1l-3.3,3.4l-2.9,5.9l-0.4,3.5l-4.5-1.5l-4.5-1.7l-7.1-0.3l-0.4,2.8l1.5,3.3l4.2-0.5
        l1.4,1.2l-2.4,7.4l2.7,3.8l0.6,4.9l-0.8,4.3l-1.7,3l-4.9-0.3l-3-3l-0.5,2.8l-3.8,0.8l-1.9,1.6l2.1,4.2l-4.3,3.5l4.6,6.7l2.2-2.7
        l1.8-1.1l2,2.2l1.5,0.6l1.9-2.4l3.1,0.1l0.4,1.8l2,1.1l3.4-4l3.3-3.1l1.4-2l-0.2-5.3l2.5-6.2l2.6-3.2l3.7-3.1l0.6-2l0.2-2.4l0.9-2.2
        l-0.3-3.6l0.7-5.6l1.1-4l1.6-3.4L1080.3,549.9L1080.3,549.9z"/>
      <path id="CO" fill={selected == 'CO' ? selectedColor : hover == 'CO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M578.3,497.2l1.2-2.1l-1.3-1.7l-2-0.4l-2.9,3.1l-2.3,1.4l-4.6,3.2l-4.3-0.5l-0.5,1.3l-3.6,0.1l-3.3,3l-1.4,5.4
        l-0.1,2.1l-2.4,0.7l-4.4,4.4l-2.9-0.2l-0.7,0.9l1.1,3.8l-1.1,1.9l-1.8-0.5l-0.9,3.1l2.2,3.4l0.6,5.4l-1.2,1.6l1.1,5.9l-1.2,3.7
        l2,1.5l-2.2,3.3l-2.5,4l-2.8,0.4l-1.4,2.3l0.2,3.2l-2.1,0.5l0.8,2l5.6,3.6l1-0.1l1.4,2.7l4.7,0.9l1.6-1l2.8,2.1l2.4,1.5l1.5-0.6
        l3.7,3l1.8,3l2.7,1.7l3.4,6.7l4.2,0.8l3-1.7l2.1,1.1l3.3-0.6l4.4,3l-3.5,6.5l1.7,0.1l2.9,3.4l2.2-17.4l0.1-2.8l-0.9-3.6l-2.5-2.4
        v-4.6l3.2-1l1.1,0.6l0.2-2.4l-3.3-0.7v-3.9l10.9,0.1l1.9-2.2l1.6,2l1,3.8l1.1-0.8l-1.7-6.4l-1.4-2.2l-2-1.4l2.9-3.1l-0.2-1.5
        l-1.5-1.9l-1-4.2l0.5-4.6l1.3-2.1l1.2-3.4l-2-1.1l-3.2,0.7l-4-0.3l-2.3,0.7l-3.8-5.5l-3.2-0.8l-7.2,0.6l-1.3-2.2l-1.3-0.6l-0.2-1.3
        l0.8-2.4l-0.4-2.5l-1.1-1.4l-0.6-2.9l-2.9-0.5l1.8-3.7l0.9-4.5l1.8-2.4l2.2-1.8l1.6-3.2L578.3,497.2L578.3,497.2z"/>
      <path id="CR" fill={selected == 'CR' ? selectedColor : hover == 'CR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M509.1,502.6l-1.4,1.3l-1.7-0.4l-0.8-1.3l-1.7-0.5l-1.4,0.8l-3.5-1.7l-0.9,0.8l-1.4,1.2l1.5,0.9l-0.9,2l-0.1,2
        l0.7,1.3l1.7,0.6l1.2,1.8l1.2-1.6l-0.3-1.8l1.4,1.1l0.3,1.9l1.9,0.8l2.1,1.3l1.5,1.5l0.1,1.4l-0.7,1.1l1.1,1.3l2.9,1.4l0.4-1.2
        l0.5-1.3l-0.1-1.2l0.8-0.7l-1.1-1l0.1-2.5l2.2-0.6l-2.4-2.7l-2-2.6L509.1,502.6L509.1,502.6z"/>
      <path id="CU" fill={selected == 'CU' ? selectedColor : hover == 'CU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M539,427.3l-4.9-2.1l-4.3-0.1l-4.7-0.5l-1.4,0.7l-4.2,0.6l-3,1.3l-2.7,1.4l-1.5,2.3l-3.1,2l2.2,0.6l2.9-0.7l0.9-1.6
        l2.3-0.1l4.4-3.3l5.4,0.3l-2.3,1.6l1.8,1.3l7,1l1.5,1.3l4.9,1.7l3.2-0.2l0.8,3.6l1.7,1.8l3.5,0.4l2.1,1.7l-4.1,3.5l7.9-0.6l3.8,0.5
        l3.7-0.3l3.8-0.8l0.8-1.5l-3.9-2.6l-4-0.3l0.6-1.7l-3.1-1.3h-1.9l-3-2.8l-4.2-4l-1.8-1.5l-5.2,0.8L539,427.3L539,427.3z"/>
      <path id="CZ" fill={selected == 'CZ' ? selectedColor : hover == 'CZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1049.4,248.5l-2.1,0.6l-1.4-0.7l-1.1,1.2l-3.4,1.2l-1.7,1.5l-3.4,1.3l1,1.9l0.7,2.6l2.6,1.5l2.9,2.6l3.8,2l2.6-2.5
        l1.7-0.5l4,1.9l2.3-0.3l2.3,1.2l0.6-1.4l2.2,0.1l1.6-0.6l0.1-0.6l0.9-0.3l0.2-1.4l1.1-0.3l0.6-1.1h1.5l-2.6-3.1l-3.6-0.3l-0.7-2
        l-3.4-0.6l-0.6,1.5l-2.7-1.2l0.1-1.7l-3.7-0.6L1049.4,248.5L1049.4,248.5z"/>
      <path id="DE" fill={selected == 'DE' ? selectedColor : hover == 'DE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1043.6,232.3l-2.4-1.9l-5.5-2.4l-2.5,1.7l-4.7,1.1l-0.1-2.1l-4.9-1.4l-0.2-2.3l-3,0.9l-3.6-0.8l0.4,3.4l1.2,2.2
        l-3,3l-1-1.3l-3.9,0.3l-0.9,1.3l1,2l-1,5.6l-1.1,2.3h-2.9l1.1,6.4l-0.4,4.2l1,1.4l-0.2,2.7l2.4,1.6l7.1,1.2l-2.3,4.2l-0.5,4.5h4.2
        l1-1.4l5.4,1.9l1.5-0.3l2.6,1.7l0.6-1.6l4.4,0.3l3.4-1.2l2.4,0.2l1.7,1.3l0.4-1.1l-1-4l1.7-0.8l1.5-2.9l-2.9-2.6l-2.6-1.5l-0.7-2.6
        l-1-1.9l3.4-1.3l1.7-1.5l3.4-1.2l1.1-1.2l1.4,0.7l2.1-0.6l-2.3-3.9l0.1-2.1l-1.4-3.3l-2-2.2l1.2-1.6L1043.6,232.3L1043.6,232.3z"/>
      <path id="DJ" fill={selected == 'DJ' ? selectedColor : hover == 'DJ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1217.8,499.2l-2.5-1.7l3.1-1.5l0.1-2.7l-1.4-1.9l-1.6,1.5l-2.4-0.5l-1.9,2.8l-1.8,3l0.5,1.7l0.2,2l3.1,0.1l1.3-0.5
        l1.3,1.1L1217.8,499.2L1217.8,499.2z"/>
      <path id="DK" fill={selected == 'DK' ? selectedColor : hover == 'DK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1035.9,221.2l-1.7-3l-6.7,2l0.9,2.5l5.1,3.4L1035.9,221.2L1035.9,221.2z M1027.3,216.1l-2.6-0.9l-0.7-1.6l1.3-2
        l-0.1-3l-3.6,1.6l-1.5,1.7l-4,0.4l-1.2,1.7l-0.7,1.6l0.4,6.1l2.1,3.4l3.6,0.8l3-0.9l-1.5-3l3.1-4.3l1.4,0.7L1027.3,216.1
        L1027.3,216.1z"/>
      <path id="DO" fill={selected == 'DO' ? selectedColor : hover == 'DO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M579.6,457.4v1.8l1.4,1l2.6-4.4l2-0.9l0.6,1.6l2.2-0.4l1.1-1.2l1.8,0.3l2.6-0.2l2.5,1.3l2.3-2.6l-2.5-2.3l-2.4-0.2
        l0.3-1.9l-3,0.1l-0.8-2.2l-1.4,0.1l-3.1-1.6l-4.4-0.1l-0.8,1.1l0.2,3.5l-0.7,2.4l-1.5,1.1l1.2,1.9L579.6,457.4L579.6,457.4z"/>
      <path id="DZ" fill={selected == 'DZ' ? selectedColor : hover == 'DZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1021,336.9l-3.6,0.4l-2.2-1.5h-5.6l-4.9,2.6l-2.7-1l-8.7,0.5l-8.9,1.2l-5,2l-3.4,2.6l-5.7,1.2l-5.1,3.5l2,4.1
        l0.3,3.9l1.8,6.7l1.4,1.4l-1,2.5l-7,1l-2.5,2.4l-3.1,0.5l-0.3,4.7l-6.3,2.5l-2.1,3.2L944,383l-5.4,1l-8.9,4.7l-0.1,7.5v0.4l-0.1,1.2
        l20.3,15.5l18.4,13.9l18.6,13.8l1.3,3l3.4,1.8l2.6,1.1l0.1,4l6.1-0.6l7.8-2.8l15.8-12.5l18.6-12.2l-2.5-4l-4.3-2.9l-2.6,1.2l-2-3.6
        l-0.2-2.7l-3.4-4.7l2.1-2.6l-0.5-4l0.6-3.5l-0.5-2.9l0.9-5.2l-0.4-3l-1.9-5.6l-2.6-11.3l-3.4-2.6v-1.5l-4.5-3.8l-0.6-4.8l3.2-3.6
        l1.1-5.3l-1-6.2L1021,336.9L1021,336.9z"/>
      <path id="EC" fill={selected == 'EC' ? selectedColor : hover == 'EC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M553.1,573.1l-2.4-1.5l-2.8-2.1l-1.6,1l-4.7-0.9l-1.4-2.7l-1,0.1l-5.6-3.6l-3.9,2.5l-3.1,1.4l0.4,2.6l-2.2,4.1
        l-1,3.9l-1.9,1l1,5.8l-1.1,1.8l3.4,2.7l2.1-2.9l1.3,2.8l-2.9,4.7l0.7,2.7l-1.5,1.5l0.2,2.3l2.3-0.5l2.3,0.7l2.5,3.2l3.1-2.6l0.9-4.3
        l3.3-5.5l6.7-2.5l6-6.7l1.7-4.1L553.1,573.1z"/>
      <path id="EG" fill={selected == 'EG' ? selectedColor : hover == 'EG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1129.7,374.8l-5.5-1.9l-5.3-1.7l-7.1,0.2l-1.8,3l1.1,2.7l-1.2,3.9l2,5.1l1.3,22.7l1,23.4h22.1h21.4h21.8l-1-1.3
        l-6.8-5.7l-0.4-4.2l1-1.1l-5.3-7l-2-3.6l-2.3-3.5l-4.8-9.9l-3.9-6.4l-2.8-6.7l0.5-0.6l4.6,9.1l2.7,2.9l2,2l1.2-1.1l1.2-3.3l0.7-4.8
        l1.3-2.5l-0.7-1.7l-3.9-9.2l0,0l-2.5,1.6l-4.2-0.4l-4.4-1.5l-1.1,2.1l-1.7-3.2l-3.9-0.8l-4.7,0.6l-2.1,1.8l-3.9,2L1129.7,374.8
        L1129.7,374.8z"/>
      <path id="ER" fill={selected == 'ER' ? selectedColor : hover == 'ER' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1198.1,474l-3.2-3.1l-1.8-5.9l-3.7-7.3l-2.6,3.6l-4,1l-1.6,2l-0.4,4.2l-1.9,9.4l0.7,2.5l6.5,1.3l1.5-4.7l3.5,2.9
        l3.2-1.5l1.4,1.3l3.9,0.1l4.9,2.5l1.6,2.2l2.5,2.1l2.5,3.7l2,2.1l2.4,0.5l1.6-1.5l-2.8-1.9l-1.9-2.2l-3.2-3.7l-3.2-3.6L1198.1,474z"
        />
      <path id="EE" fill={selected == 'EE' ? selectedColor : hover == 'EE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1093.2,197.5l-5.5,0.9l-5.4,1.6l0.9,3.4l3.3,2.1l1.5-0.8l0.1,3.5l3.7-1l2.1,0.7l4.4,2.2h3.8l1.6-1.9l-2.5-5.5
        l2.6-3.4l-0.9-1l0,0l-4.6,0.2L1093.2,197.5z"/>
      <path id="ET" fill={selected == 'ET' ? selectedColor : hover == 'ET' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1187.6,477l-1.5,4.7l-6.5-1.3l-0.7,5.5l-2.1,6.2l-3.2,3.2l-2.3,4.8l-0.5,2.6l-2.6,1.8l-1.4,6.7v0.7l0.2,5l-0.8,2
        l-3,0.1l-1.8,3.6l3.4,0.5l2.9,3.1l1,2.5l2.6,1.5l3.5,6.9l2.9,1.1v3.6l2,2.1h3.9l7.2,5.4h1.8l1.3-0.1l1.2,0.7l3.8,0.5l1.6-2.7
        l5.1-2.6l2.3,2.1h3.8l1.5-2l3.6-0.1l4.9-4.5l7.4-0.3l15.4-19.1l-4.8,0.1l-18.5-7.6l-2.2-2.2l-2.1-3.1l-2.2-3.5l1.1-2.3l-1.3-1.1
        l-1.3,0.5l-3.1-0.1l-0.2-2l-0.5-1.7l1.8-3l1.9-2.8l-2-2.1l-2.5-3.7l-2.5-2.1l-1.6-2.2l-4.9-2.5l-3.9-0.1l-1.4-1.3l-3.2,1.5
        L1187.6,477L1187.6,477z"/>
      <path id="FI" fill={selected == 'FI' ? selectedColor : hover == 'FI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1093.4,144.4l0.8-3.8l-5.7-2.1l-5.8,1.8l-1.1,3.9l-3.4,2.4l-4.7-1.3l-5.3,0.3l-5.1-2.9l-2.1,1.4l5.9,2.7l7.2,3.7
        l1.7,8.4l1.9,2.2l6.4,2.6l0.9,2.3l-2.6,1.2l-8.7,6.1l-3.3,3.6l-1.5,3.3l2.9,5.2l-0.1,5.7l4.7,1.9l3.1,3.1l7.1-1.2l7.5-2.1l8-0.5l0,0
        l7.9-7.4l3.3-3.3l0.9-2.9l-7.3-3.9l0.9-3.7l-4.9-4.1l1.7-4.8l-6.4-6.3l2.8-4.1l-7.2-3.7L1093.4,144.4L1093.4,144.4z"/>
      <path id="FJ" fill={selected == 'FJ' ? selectedColor : hover == 'FJ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1976.7,674.4l-3.7,2l-1.9,0.3l-3.1,1.3l0.2,2.4l3.9-1.3l3.9-1.6L1976.7,674.4L1976.7,674.4z M1965.7,682.5l-1.6,1
        l-2.3-0.8l-2.7,2.2l-0.2,2.8l2.9,0.8l3.6-0.9l1.8-3.3L1965.7,682.5L1965.7,682.5z"/>
      <path id="GA" fill={selected == 'GA' ? selectedColor : hover == 'GA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1050.2,557.7l-0.7-0.3l-3.4,0.8l-3.4-0.8l-2.6,0.4v7.6h-8.2l-1.9,0.3l-1.1,4.8l-1.3,4.6l-1.3,2l-0.2,2.1l3.4,6.6
        l3.7,5.3l5.8,6.4l4.3-3.5l-2.1-4.2l1.9-1.6l3.8-0.8l0.5-2.8l3,3l4.9,0.3l1.7-3l0.8-4.3l-0.6-4.9l-2.7-3.8l2.4-7.4l-1.4-1.2l-4.2,0.5
        l-1.5-3.3L1050.2,557.7L1050.2,557.7z"/>
      <path id="GB" fill={selected == 'GB' ? selectedColor : hover == 'GB' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M950,227.5l-4.9-3.7l-3.9,0.3l0.8,3.2l-1.1,3.2l2.9-0.1l3.5,1.3L950,227.5z M963,203.2l-5.5,0.5l-3.6-0.4l-3.7,4.8
        l-1.9,6.1l2.2,3l0.1,5.8l2.6-2.8l1.4,1.6l-1.7,2.7l1,1.6l5.7,1.1h0.1l3.1,3.8l-0.8,3.5l0,0l-7.1-0.6l-1,4l2.6,3.3l-5.1,1.9l1.3,2.4
        l7.5,1l0,0l-4.3,1.3l-7.3,6.5l2.5,1.2l3.5-2.3l4.5,0.7l3.3-2.9l2.2,1.2l8.3-1.7l6.5,0.1l4.3-3.3l-1.9-3.1l2.4-1.8l0.5-3.9l-5.8-1.2
        l-1.3-2.3l-2.9-6.9l-3.2-1l-4.1-7.1l-0.4-0.6l-4.8-0.4l4.2-5.3l1.3-4.9h-5l-4.7,0.8L963,203.2L963,203.2z"/>
      <path id="GE" fill={selected == 'GE' ? selectedColor : hover == 'GE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1200,300.2l-7.5-2.9l-7.7-1l-4.5-1.1l-0.5,0.7l2.2,1.9l3,0.7l3.4,2.3l2.1,4.2l-0.3,2.7l5.4-0.3l5.6,3l6.9-1l1.1-1
        l4.2,1.8l2.8,0.4l0.6-0.7l-3.2-3.4l1.1-0.9l-3.5-1.4l-2.1-2.5l-5.1-1.3l-2.9,1L1200,300.2L1200,300.2z"/>
      <path id="GH" fill={selected == 'GH' ? selectedColor : hover == 'GH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M976.8,502.1l-2.6-0.5l-1.8,1l-2.4-0.5l-9.7,0.3l-0.2,3.6l0.8,4.8l1.4,9.1l-2.3,5.3l-1.5,7.2l2.4,5.5l-0.2,2.5
        l5,1.8l5-1.9l3.2-2.1l8.7-3.8l-1.2-2.2l-1.5-4l-0.4-3.2l1.2-5.7l-1.4-2.3l-0.6-5.1l0.1-4.6l-2.4-3.3L976.8,502.1L976.8,502.1z"/>
      <path id="GN" fill={selected == 'GN' ? selectedColor : hover == 'GN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M912.4,493l-0.8,0.4l-3-0.5l-0.4,0.7l-1.3,0.1l-4-1.5l-2.7-0.1l-0.1,2.1l-0.6,0.7l0.4,2.1l-0.8,0.9h-1.3l-1.4,1
        l-1.7-0.1l-2.6,3.1l1.6,1.1l0.8,1.4l0.7,2.8l1.3,1.2l1.5,0.9l2.1,2.5l2.4,3.7l3-2.8l0.7-1.7l1-1.4l1.5-0.2l1.3-1.2h4.5l1.5,2.3
        l1.2,2.7L917,515l0.9,1.7v2.3l1.5-0.3l1.2-0.2l1.5-0.7l2.3,3.9l-0.4,2.6l1.1,1.3l1.6,0.1l1.1-2.6l1.6,0.2h0.9l0.3-2.8l-0.4-1.2
        l0.6-0.9l2-0.8l-1.3-5.1l-1.3-2.6l0.5-2.2l1.1-0.5l-1.7-1.8l0.3-1.9l-0.7-0.7l-1.2,0.6l0.2-2.1l1.2-1.6l-2.3-2.7l-0.6-1.7l-1.3-1.4
        l-1.1-0.2l-1.3,0.9l-1.8,0.8l-1.6,1.4l-2.4-0.5l-1.5-1.6l-0.9-0.2l-1.5,0.8h-0.9L912.4,493L912.4,493z"/>
      <path id="GM" fill={selected == 'GM' ? selectedColor : hover == 'GM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M882.8,488.5l5,0.1l1.4-0.9h1l2.1-1.5l2.4,1.4l2.4,0.1l2.4-1.5l-1.1-1.8l-1.8,1.1l-1.8-0.1l-2.1-1.5l-1.8,0.1
        l-1.3,1.5l-6.1,0.2L882.8,488.5L882.8,488.5z"/>
      <path id="GW" fill={selected == 'GW' ? selectedColor : hover == 'GW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M900.2,492.1l-10.3-0.3l-1.5,0.7l-1.8-0.2l-3,1.1l0.3,1.3l1.7,1.4v0.9l1.2,1.8l2.4,0.5l2.9,2.6l2.6-3.1l1.7,0.1
        l1.4-1h1.3l0.8-0.9l-0.4-2.1l0.6-0.7L900.2,492.1L900.2,492.1z"/>
      <path id="GQ" fill={selected == 'GQ' ? selectedColor : hover == 'GQ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1040.1,557.8l-9.2-0.2l-1.9,7.2l1,0.9l1.9-0.3h8.2V557.8L1040.1,557.8z"/>
      <path id="GR" fill={selected == 'GR' ? selectedColor : hover == 'GR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1101.9,344.9l-0.8,2.8l6.6,1.2v1.1l7.6-0.6l0.5-1.9l-2.8,0.8v-1.1l-3.9-0.5l-4.1,0.4L1101.9,344.9L1101.9,344.9z
        M1113.4,307.5l-2.7-1.6l0.3,3l-4.6,0.6l-3.9-2.1l-3.9,1.7l-3.8-0.2l-1,0.2l-0.7,1.1l-2.8-0.1l-1.9,1.3l-3.3,0.6v1.6l-1.6,0.9
        l-0.1,2.1l-2.1,3l0.5,1.9l2.9,3.6l2.3,3l1.3,4.3l2.3,5.1l4.6,2.9l3.4-0.1l-2.4-5.7l3.3-0.7l-1.9-3.3l5,1.7l-0.4-3.7l-2.7-1.8l-3.2-3
        l1.8-1.4l-2.8-3l-1.6-3.8l0.9-1.3l3,3.2h2.9l2.5-1l-3.9-3.6l6.1-1.6l2.7,0.6l3.2,0.2l1.1-0.7L1113.4,307.5L1113.4,307.5z"/>
      <path id="GL" fill={selected == 'GL' ? selectedColor : hover == 'GL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M887.4,76.3l-26-0.4l-11.8,0.3l-5,1.3l-11.5-0.1l-12.7,2.1l-1.6,1.7l6.7,2.1l-6.2-1.3l-4.5-0.3l-7-1.4l-10.6,2.1
        l-2.7-1.2h-10.4l-10.9,0.6l-8.9,1l-0.2,1.8l-5.3,0.5L744.2,88l-4.6,1.7l8.1,1.5l-2.8,1.6L730,95l-15.5,2.2l-2.2,1.7l6.4,2l14.5,1.2
        l-7.5,0.2l-10.9,1.5l3.8,3.1l3,1.5l9.4-0.3l10.1-0.2l7.6,0.3l8,2.9l-1.4,2.1l3.6,1.9l1.4,5.3l1,3.6l1.4,1.9l-7,4.8l2.6,1.3l4.4-0.8
        l2.6,1.8l5.3,3.4l-7.5-1.4h-3.8l-3,2.8l-1.5,3.6l4.2,1.8l4-0.8l2.6-0.8l5.5-1.9l-2.8,4.2l-2.6,2.3l-7.1,2l-7,6.3l2,2l-3.4,4l3.7,5.2
        l-1.5,5l0.7,3.7l4.8,7.1l0.8,5.6l3.1,3.2h8.9l5,4.7l6.5-0.3l4.1-5.7l3.5-4.8l-0.3-4.4l8.6-4.6l3.3-3.7l1.4-3.9l4.7-3.5l6.5-1.3
        l6.1-1.4l3-0.2l10.2-3.9l7.4-5.7l4.8-2.1l4.6-0.1l12.5-1.8l12.1-4.3l11.9-4.6l-5.5-0.3l-10.6-0.2l5.3-2.8l-0.5-3.6l4.2,3l2.7,2.1
        l7.3-1l-0.6-4.3l-4.5-3.1l-5-1.3l2.4-1.4l7.2,2.1l0.5-2.3l-4.1-3.4h5.4l5.6-0.8l1.7-1.8l-4-2.1l8.6-0.3l-4-4.3l4.1-0.5l0.1-4.2
        l-6.2-2.5l6.4-1.6l5.8-0.1l-3.6-3.2l1.1-5.1l3.6-2.9l4.9-3.2l-8-0.2l11.3-0.7l2.2-1l14.6-2.9l-1.6-1.7l-10-0.8l-16.9,1.5l-9.2,1.5
        l4.5-2.3l-2.3-1.4l-7,1.2l-9.7-1.4l-12.1,0.5l-1.4-0.7l18.3-0.4l12.9-0.2l6.6-1.4L887.4,76.3L887.4,76.3z"/>
      <path id="GT" fill={selected == 'GT' ? selectedColor : hover == 'GT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M482.8,458.9l-5.1-0.1h-5.2l-0.4,3.6h-2.6l1.8,2.1l1.9,1.5l0.5,1.4l0.8,0.4l-0.4,2.1H467l-3.3,5.2l0.7,1.2l-0.8,1.5
        l-0.4,1.9l2.7,2.6l2.5,1.3l3.4,0.1l2.8,1.1l0.2-1l2.1-1.6l1.1-0.7l-0.2-0.7l1.4-0.4l1.3-1.6l-0.3-1.3l0.5-1.2l2.8-1.8l2.8-2.4
        l-1.5-0.8l-0.6,0.9l-1.7-1.1h-1.6l1.2-7.2L482.8,458.9L482.8,458.9z"/>
      <path id="GY" fill={selected == 'GY' ? selectedColor : hover == 'GY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M656.1,534.2l-2.1-2.3l-2.9-3.1l-2.1-0.1l-0.1-3.3l-3.3-4.1l-3.6-2.4l-4.6,3.8l-0.6,2.3l1.9,2.3l-1.5,1.2l-3.4,1.1
        v2.9l-1.6,1.8l3.7,4.8l2.9-0.3l1.3,1.5l-0.8,2.8l1.9,0.9l1.2,3l-1.6,2.2l-1,5.4l1.4,3.3l0.3,2.9l3.5,3l2.7,0.3l0.7-1.3l1.7-0.2
        l2.6-1.1l1.8-1.7l3.1,0.5l1.4-0.2l-3.3-5.6L655,551l-1.8-0.1l-2.4-4.6l1.1-3.3l-0.3-1.5l3.5-1.6L656.1,534.2L656.1,534.2z"/>
      <path id="HN" fill={selected == 'HN' ? selectedColor : hover == 'HN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M514.1,476.8l-1.3-1.8l-1.9-1l-1.5-1.4l-1.6-1.2l-0.8-0.1l-2.5-0.9l-1.1,0.5l-1.5,0.2l-1.3-0.4l-1.7-0.4l-0.8,0.7
        l-1.8,0.7l-2.6,0.2l-2.5-0.6l-0.9,0.4l-0.5-0.6l-1.6,0.1l-1.3,1.1l-0.6-0.2l-2.8,2.4l-2.8,1.8l-0.5,1.2l0.3,1.3l-1.3,1.6l1.5,0.5
        l1.1,1.3l1.6,1l0.1,0.9l2.5-0.8l1.1,0.5l0.7,0.7l-0.6,2.5l1.7,0.6l0.7,2l1.8-0.3l0.8-1.5h0.8l0.2-3.1l1.3-0.2h1.2l1.4-1.7l1.5,1.3
        l0.6-0.8l1.1-0.7l2.1-1.8l0.3-1.3l0.5,0.1l0.8-1.5l0.6-0.2l0.9,0.9l1.1,0.3l1.3-0.8h1.4l2-0.8l0.9-0.9L514.1,476.8L514.1,476.8z"/>
      <path id="HR" fill={selected == 'HR' ? selectedColor : hover == 'HR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1065,280.4l-4-2.6l-1.6-0.8l-3.9,1.7l-0.3,2.5l-1.7,0.6l0.2,1.7l-2-0.1l-1.8-1l-0.8,1l-3.5-0.2l-0.2,0.1v2.2l1.7,2
        l1.3-2.6l3.3,1l0.3,2l2.5,2.6l-1,0.5l4.6,4.5l4.8,1.8l3.1,2.2l5,2.3l0,0l0.5-1l-4.7-2.4l-2.2-2.5l-2-1.4l-2.5-2.3l-1.3-1.9l-2.7-2.9
        l0.9-2.5l1.9,1.4l1-1.3l2.3-0.1l4.4,1l3.5-0.1l2.4,1.4l0,0l1.7-2.3l-1.7-1.8l-1.5-2.4l0,0l-1.8,0.9L1065,280.4L1065,280.4z"/>
      <path id="HT" fill={selected == 'HT' ? selectedColor : hover == 'HT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M580.6,446.7l-4.6-1l-3.4-0.2l-1.4,1.7l3.4,1l-0.3,2.4l2.2,2.8l-2.1,1.4l-4.2-0.5l-5-0.9l-0.7,2.1l2.8,1.9l2.7-1.1
        l3.3,0.4l2.7-0.4l3.6,1.1l0.2-1.8l-1.2-1.9l1.5-1.1l0.7-2.4L580.6,446.7z"/>
      <path id="HU" fill={selected == 'HU' ? selectedColor : hover == 'HU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1079.1,263.8l-1.6,0.4l-1,1.5l-2.2,0.7l-0.6-0.4l-2.3,1l-1.9,0.2l-0.3,1.2l-4.1,0.8l-1.9-0.7l-2.6-1.6l-0.2,2.6
        h-2.8l1.1,1.3l-1.3,4l0.8,0.1l1.2,2.1l1.6,0.8l4,2.6l4.2,1.2l1.8-0.9l0,0l3.7-1.6l3.2,0.2l3.8-1.1l2.6-4.3l1.9-4.2l2.9-1.3l-0.6-1.6
        l-2.9-1.7l-1,0.6L1079.1,263.8L1079.1,263.8z"/>
      <path id="ID" fill={selected == 'ID' ? selectedColor : hover == 'ID' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1651.9,637.3l0.5-1.7l-1.8-1.9l-2.8-2l-5.3,1.3l7,4.4L1651.9,637.3L1651.9,637.3z M1672.8,636.7l4-4.8l0.1-1.9
        l-0.5-1.3l-5.7,2.6l-2.8,3.9l-0.7,2.1l0.6,0.8L1672.8,636.7L1672.8,636.7z M1637.2,623.7l-1.6,2.2l-3.1,0.1l-2.2,3.6l3,0.1l3.9-0.9
        l6.6-1.2l-1.2-2.8l-3.5,0.6L1637.2,623.7L1637.2,623.7z M1665.3,623.7l-5.2,2.3l-3.8,0.5l-3.4-1.9l-4.5,1.3l-0.2,2.3l7.4,0.8
        l8.6-1.8L1665.3,623.7L1665.3,623.7z M1585.8,615.3l-0.7-2.3l-2.3-0.5l-4.4-2.4l-6.8-0.4l-4.1,6.1l5.1,0.4l0.8,2.8l10,2.6l2.4-0.8
        l4.1,0.6l6.3,2.4l5.2,1.2l5.8,0.5l5.1-0.2l5.9,2.5l6.6-2.4l-6.6-3.8l-8.3-1.1l-1.8-4.1l-10.3-3.1l-1.3,2.6L1585.8,615.3
        L1585.8,615.3z M1732.4,611.7l0.2-3l-1.2-1.9l-1.3,2.2l-1.2,2.2l0.3,4.8L1732.4,611.7z M1691.4,594.2l-1.4-2.1l-5.7,0.3l1,2.7
        l3.9,1.2L1691.4,594.2L1691.4,594.2z M1709.5,591.8l-6.1-1.8l-6.9,0.3l-1.5,3.5l3.9,0.2l3.2-0.4l4.6,0.5l4.7,2.6L1709.5,591.8
        L1709.5,591.8z M1730.5,579.5l-0.8-2.4l-9-2.6l-2.9,2.1l-7.6,1.5l2.3,3.2l5,1.2l2.1,3.7l8.3,0.1l0.4,1.6l-4-0.1l-6.2,2.3l4.2,3.1
        l-0.1,2.8l1.2,2.3l2.1-0.5l1.8-3.1l8.2,5.9l4.6,0.5l10.6,5.4l2.3,5.3l1,6.9l-3.7,1.8l-2.8,5.2l7.1-0.2l1.6-1.8l5.5,1.3l4.6,5.2
        l1.5-20.8l1-20.7l-6-1.2l-4.1-2.3l-4.7-2.2h-5l-6.6,3.8l-4.9,6.8l-5.7-3.8L1730.5,579.5z M1680.5,563.1l-1-1.4l-5.5,4.6l-6.5,0.3
        l-7.1-0.9l-4.4-1.9l-4.7,4.8l-1.2,2.6l-2.9,9.6l-0.9,5l-2.4,4.2l1.6,4.3l2.3,0.1l0.6,6.1l-1.9,5.9l2.3,1.9l3.6-1l0.3-9.1l-0.2-7.4
        l3.8-1.9l-0.7,6.2l3.9,3.7l-0.8,2.5l1.3,1.7l5.6-2.4l-3,5.2l2.1,2.2l3.1-1.9l0.3-4.1l-4.7-7.4l1.1-2.2l-5.1-8.1l5-2.5l2.6-3.7
        l2.4,0.9l0.5-2.9l-10.5,2.1l-3.1,2.9l-5-5.6l0.9-4.8l4.9-1l9.3-0.3l5.4,1.3l4.3-1.3L1680.5,563.1L1680.5,563.1z M1699.9,565
        l-0.6-2.6l-3.3-0.6l-0.5-3.5l-1.8,2.3l-1,5.1l1.7,8.2l2.2,4l1.6-0.8l-2.3-3.3l0.9-3.9l2.9,0.6L1699.9,565L1699.9,565z M1639,560.5
        l0.9-2.9l-4.3-6l3-5.8l-5-1h-6.4l-1.7,7.2l-2,2.2l-2.7,8.9l-4.5,1.3l-5.4-1.8l-2.7,0.6l-3.2,3.2l-3.6-0.4l-3.6,1.2l-3.9-3.5l-1-4.3
        l-3.3,4.2l-0.6,5.9l0.8,5.6l2.6,5.4l2.8,1.8l0.7,8.5l4.6,0.8l3.6-0.4l2,3.1l6.7-2.3l2.8,2l4,0.4l2,3.9l6.5-2.9l0.8,2.3l2.5-9.7
        l0.3-6.4l5.5-4.3l-0.2-5.8l1.8-4.3l6.7-0.8L1639,560.5L1639,560.5z M1570.3,609.4l0.7-9.8l1.7-8l-2.6-4l-4.1-0.5l-1.9-3.6l-0.9-4.4
        l-2-0.2l-3.2-2.2l2.3-5.2l-4.3-2.9l-3.3-5.3l-4.8-4.4l-5.7-0.1l-5.5-6.8l-3.2-2.7l-4.5-4.3l-5.2-6.2l-8.8-1.2l-3.6-0.3l0.6,3.2
        l6.1,7l4.4,3.6l3.1,5.5l5.1,4l2.2,4.9l1.7,5.5l4.9,5.3l4.1,8.9l2.7,4.8l4.1,5.2l2.2,3.8l7,5.2l4.5,5.3L1570.3,609.4L1570.3,609.4z"
        />
      <path id="IN" fill={selected == 'IN' ? selectedColor : hover == 'IN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1414.1,380.1l-8.5-4.4l-6.2-4l-3.2-7l4.1,0.9l-0.6-3.3l-3-3.3l-0.8-5.2l-7.6-7.5l-3.7,5.4l-5.7,1l-8.5-1.6
        l-1.9,2.8l3.2,5.6l2.9,4.3l5,3.1l-3.7,3.7l1,4.5l-3.9,6.3l-2.1,6.5l-4.5,6.7l-6.4-0.5l-4.9,6.6l4,2.9l1.3,4.9l3.5,3.2l1.8,5.5h-12
        l-3.2,4.2l7.1,5.4l1.9,2.5l-2.4,2.3l8,7.7l4,0.8l7.6-3.8l1.7,5.9l0.8,7.8l2.5,8.1l3.6,12.3l5.8,8.8l1.3,3.9l2,8l3.4,6.1l2.2,3
        l2.5,6.4l3.1,8.9l5.5,6l2.2-1.8l1.7-4.4l5-1.8l-1.8-2.1l2.2-4.8l2.9-0.3l-0.7-10.8l1.9-6.1l-0.7-5.3l-1.9-8.2l1.2-4.9l2.5-0.3
        l4.8-2.3l2.6-1.6l-0.3-2.9l5-4.2l3.7-4l5.3-7.5l7.4-4.2l2.4-3.8l-0.9-4.8l6.6-1.3l3.7,0.1l0.5-2.4l-1.6-5.2l-2.6-4.8l0.4-3.8
        l-3.7-1.7l0.8-2.3l3.1-2.4l-4.6-3.4l1.2-4.3l4.8,2.7l2.7,0.4l1.2,4.4l5.4,0.9l5-0.1l3.4,1.1l-1.6,5.3l-2.4,0.4l-1.1,3.6l3.5,3.3
        l0.2-4l1.5-0.1l4.5,10.1l2.4-1.5l-0.9-2.7l0.9-2.1l-0.9-6.6l4.6,1.4l1.5-5.2l-0.3-3.1l2.1-5.4l-0.9-3.6l6.1-4.4l4.1,1.1l-1.3-3.9
        l1.6-1.2l-0.9-2.4l-6.1-0.9l1.2-2.7l-3.5-3.9l-3.2,2.6l-4.9-1.5l-5.3,4l-3.9,4.8l-4.2,0.8l2.7,2l0.4,3.9l-4.4,0.2l-4.7-0.4l-3.2,1
        l-5.5-2.5l-0.3-1.2l-1.5-5.1l-3,1.4l0.1,2.7l1.5,4.1l-0.1,2.5l-4.6,0.1l-6.8-1.5l-4.3-0.6l-3.8-3.2l-7.6-0.9l-7.7-3.5l-5.8-3.1
        l-5.7-2.5l0.9-5.9L1414.1,380.1L1414.1,380.1z"/>
      <path id="IE" fill={selected == 'IE' ? selectedColor : hover == 'IE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M947.3,231.7l-3.5-1.3l-2.9,0.1l1.1-3.2l-0.8-3.2l-3.7,2.8l-6.7,4.7l2.1,6.1l-4.2,6.4l6.7,0.9l8.7-3.6l3.9-5.4
        L947.3,231.7L947.3,231.7z"/>
      <path id="IR" fill={selected == 'IR' ? selectedColor : hover == 'IR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1213.5,324.4l-3.2-2.9l-1.2-2.4l-3.3,1.8l2.9,7.3l-0.7,2l3.7,5.2l0,0l4.7,7.8l3.7,1.9l1,3.8l-2.3,2.2l-0.5,5
        l4.6,6.1l7,3.4l3.5,4.9l-0.2,4.6h1.7l0.5,3.3l3.4,3.4l1.7-2.5l3.7,2.1l2.8-1l5.1,8.4l4.3,6.1l5.5,1.8l6.1,4.9l6.9,2.1l5.1-3.1l4-1.1
        l2.8,1.1l3.2,7.8l6.3,0.8l6.1,1.5l10.5,1.9l1.2-7.4l7.4-3.3l-0.9-2.9l-2.7-1l-1-5.7l-5.6-2.7l-2.8-3.9l-3.2-3.3l3.9-5.8l-1.1-4
        l-4.3-1.1l-1.1-4l-2.7-5.1l1.6-3.5l-2.5-0.9l0.5-4.7l0.5-8l-1.6-5.5l-3.9-0.2l-7.3-5.7l-4.3-0.7l-6.5-3.3l-3.8-0.6l-2.1,1.2
        l-3.5-0.2l-3,3.7l-4.4,1.2l-0.2,1.6l-7.9,1.7l-7.6-1.1l-4.3-3.3l-5.2-1.3l-2.5-4.8l-1.3,0.3l-3.8-3.4l1.2-3.1l-1.9-1.9l-1.9,0.5
        l-5.3,4.7l-1.8,0.2L1213.5,324.4L1213.5,324.4z"/>
      <path id="IQ" fill={selected == 'IQ' ? selectedColor : hover == 'IQ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1207.3,334.9l-6.2-0.9l-2.1,1l-2.1,4.1l-2.7,1.6l1.2,4.7l-0.9,7.8l-11,6.7l3.1,7.7l6.7,1.7l8.5,4.5l16.7,12.7
        l10.2,0.5l3.2-6.1l3.7,0.5l3.2,0.4l-3.4-3.4l-0.5-3.3h-1.7l0.2-4.6l-3.5-4.9l-7-3.4l-4.6-6.1l0.5-5l2.3-2.2l-1-3.8l-3.7-1.9
        l-4.7-7.8l0,0l-2.3,1.1L1207.3,334.9L1207.3,334.9z"/>
      <path id="IS" fill={selected == 'IS' ? selectedColor : hover == 'IS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M915.7,158.6l-6.9-0.4l-7.3,2.9l-5.1-1.5l-6.9,3l-5.9-3.8l-6.5,0.8l-3.6,3.7l8.7,1.3l-0.1,1.6l-7.8,1.1l8.8,2.7
        l-4.6,2.5l11.7,1.8l5.6,0.8l3.9-1l12.9-3.9l6.1-4.2l-4.4-3.8L915.7,158.6L915.7,158.6z"/>
      <path id="IL" fill={selected == 'IL' ? selectedColor : hover == 'IL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1167.8,360.5l-1.4,0.1l-0.4,1.1h-1.8l-0.1,0.1l-0.6,1.6l-0.6,4.8l-1.1,2.9l0.4,0.4l-1.4,2.1l0,0l3.9,9.2l0.7,1.7
        l1.7-10.2l-0.4-2.4l-2.4,0.8l0.1-1.7l1.2-0.8l-1.4-0.7l0.7-4.3l2,0.9l0.7-2h-0.1l0.6-1L1167.8,360.5L1167.8,360.5z"/>
      <path id="IT" fill={selected == 'IT' ? selectedColor : hover == 'IT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1057.8,328.6l-1.6,5.1l0.9,2l-0.9,3.3l-4.2-2.4l-2.7-0.7l-7.5-3.3l0.6-3.4l6.2,0.6l5.2-0.7L1057.8,328.6z
        M1072.3,316.2l-0.8,2.3l-3.1-3l-4.5-1l-1.9,4.1l3.9,2.3l-0.4,3.3l-2.1,0.4l-2.5,5.6l-2.1,0.5l-0.1-2l0.8-3.5l1.1-1.3l-2.3-3.7
        l-1.8-3.2l-2.2-0.8l-1.7-2.7l-3.4-1.2l-2.3-2.5l-3.9-0.4l-4.2-2.8l-4.9-4l-3.6-3.6l-1.9-6l-2.6-0.7l-4.2-2.1l-2.3,0.9l-2.8,2.8
        l-2.1,0.5l0.5-2.7l-2.7-0.8l-1.5-4.8l1.7-1.8l-1.6-2.4l0.2-1.7l2.2,1.3l2.4-0.3l2.7-2.1l0.9,1l2.4-0.2l0.9-2.5l3.8,0.8l2.1-1.1
        l0.3-2.5l3.1,0.9l0.5-1.2l4.8-1.1l1.3,2.2l7.2,1.6l-0.3,3l1.4,2.7l-4.1-0.9l-3.9,2.2l0.4,3l-0.5,1.8l1.9,3.1l4.9,3.1l2.9,5.1l6,5
        l4-0.1l1.4,1.4l-1.4,1.2l4.8,2.3l3.9,1.9l4.7,3.2L1072.3,316.2z M1040.2,305.3l-0.1-0.6l-0.6,0.1l-0.2,0.5H1040.2z M1040.3,292.4
        l-0.9,0.3l0.2,0.9l0.7-0.1L1040.3,292.4z M1021.6,311.6l-2.8-0.3l1.3,3.6l0.4,7.6l2.1,1.7l2-2.1l2.4,0.4l0.4-8.4l-3.3-4.4
        L1021.6,311.6z"/>
      <path id="VA" fill={selected == 'VA' ? selectedColor : hover == 'VA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1039.5,304.8l0.6-0.1l0.1,0.6h-0.9L1039.5,304.8z"/>
      <path id="SM" fill={selected == 'SM' ? selectedColor : hover == 'SM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1040.3,293.5l-0.7,0.1l-0.2-0.9l0.9-0.3L1040.3,293.5z"/>
      <path id="JM" fill={selected == 'JM' ? selectedColor : hover == 'JM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M550.7,458.5l3.9-0.1l-0.8-1.8l-2.7-1.5l-3.7-0.6l-1.2-0.2l-2.4,0.4l-0.8,1.5l2.9,2.3l3,1L550.7,458.5L550.7,458.5z
        "/>
      <path id="JO" fill={selected == 'JO' ? selectedColor : hover == 'JO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1186.6,367.6l-3.1-7.7l-9.6,6.7l-6.3-2.5l-0.7,2l0.4,3.9l-0.6,1.9l0.4,2.4l-1.7,10.2l0.3,0.9l6.1,1l2.1-2l1.1-2.3
        l4-0.8l0.7-2.2l1.7-1l-6.1-6.4l10.4-3.1L1186.6,367.6L1186.6,367.6z"/>
      <path id="JP" fill={selected == 'JP' ? selectedColor : hover == 'JP' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1692.5,354.9l-4.5-1.3l-1.1,2.7l-3.3-0.8l-1.3,3.8l1.2,3l4.2,1.8l-0.1-3.7l2.1-1.5l3.1,2.1l1.3-3.9L1692.5,354.9
        L1692.5,354.9z M1716.9,335.6l-3.6-6.7l1.3-6.4l-2.8-5.2l-8.1-8.7l-4.8,1.2l0.2,3.9l5.1,7.1l1,7.9l-1.7,2.5l-4.5,6.5l-5-3.1v11.5
        l-6.3-1.3l-9.6,1.9l-1.9,4.4l-3.9,3.3l-1.1,4l-4.3,2l4,4.3l4.1,1.9l0.9,5.7l3.5,2.5l2.5-2.7l-0.8-10.8l-7.3-4.7l6.1-0.1l5-3l8.6-1.4
        l2.4,4.8l4.6,2.4l4.4-7.3l9.1-0.4l5.4-3l0.6-4.6l-2.5-3.2L1716.9,335.6L1716.9,335.6z M1705.1,291.4l-5.3-2.1l-10.4-6.4l1.9,4.8
        l4.3,8.5l-5.2,0.4l0.6,4.7l4.6,6.1h5.7l-1.6-6.8l10.8,4.2l0.4-6.1l6.4-1.7l-6-6.9l-1.7,2.6L1705.1,291.4L1705.1,291.4z"/>
      <path id="KZ" fill={selected == 'KZ' ? selectedColor : hover == 'KZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1308.8,223.8l-9-1.3l-3.1,2.5l-10.8,2.2l-1.7,1.5l-16.8,2.1l-1.4,2.1l5,4.1l-3.9,1.6l1.5,1.7l-3.6,2.9l9.4,4.2
        l-0.2,3l-6.9-0.3l-0.8,1.8l-7.3-3.2l-7.6,0.2l-4.3,2.5l-6.6-2.4l-11.9-4.3l-7.5,0.2l-8.1,6.6l0.7,4.6l-6-3.6l-2.1,6.8l1.7,1.2
        l-1.7,4.7l5.3,4.3l3.6-0.2l4.2,4.1l0.2,3.2l2.8,1l4.4-1.3l5-2.7l4.7,1.5l4.9-0.3l1.9,3.9l0.6,6l-4.6-0.9l-4,1l0.9,4.5l-5-0.6l0.6,2
        l3.2,1.6l3.7,5.5l6.4,2.1l1.5,2.1l-0.7,2.6l0.7,1.5l1.8-2l5.5-1.3l3.8,1.7l4.9,4.9l2.5-0.3l-6.2-22.8l11.9-3.6l1.1,0.5l9.1,4.5
        l4.8,2.3l6.5,5.5l5.7-0.9l8.6-0.5l7.5,4.5l1.5,6.2l2.5,0.1l2.6,5l6.6,0.2l2.3,3h1.9l0.9-4.5l5.4-4.3l2.5-1.2l0.3-2.7l3.1-0.8
        l9.1,2.1l-0.5-3.6l2.5-1.3l8.1,2.6l1.6-0.7l8.6,0.2l7.8,0.6l3.3,2.2l3.5,0.9l-1.7-3.5l2.9-1.6l-8.7-10.7l9-2.4l2-1.4l-1-11.1l10.7,2
        l1.6-2.8l-2.5-6.2l3.8-0.6l1.8-4.2l-4.3-3.8l-6,0.9l-3.3-2.6l-3.9-1.2l-4.1-3.6l-3.2-1.1l-6.2,1.6l-8.3-3.6l-1.1,3.3l-18.1-15.5
        l-8.3-4.7l0.8-1.9l-9.1,5.7l-4.4,0.4l-1.2-3.3l-7-2.1l-4.3,1.5L1308.8,223.8L1308.8,223.8z"/>
      <path id="KE" fill={selected == 'KE' ? selectedColor : hover == 'KE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1211.7,547.2h-3.8l-2.3-2.1l-5.1,2.6l-1.6,2.7l-3.8-0.5l-1.2-0.7l-1.3,0.1h-1.8l-7.2-5.4h-3.9l-2-2.1v-3.6
        l-2.9-1.1l-3.8,4.2l-3.4,3.8l2.7,4.4l0.7,3.2l2.6,7.3l-2.1,4.7l-2.7,4.2l-1.6,2.6v0.3l1.4,2.4l-0.4,4.7l20.2,13l0.4,3.7l8,6.3
        l2.2-2.1l1.2-4.2l1.8-2.6l0.9-4.5l2.1-0.4l1.4-2.7l4-2.5l-3.3-5.3l-0.2-23.2L1211.7,547.2L1211.7,547.2z"/>
      <path id="KG" fill={selected == 'KG' ? selectedColor : hover == 'KG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1387.2,302.6l-3.5-0.9l-3.3-2.2l-7.8-0.6l-8.6-0.2l-1.6,0.7l-8.1-2.6l-2.5,1.3l0.5,3.6l-9.1-2.1l-3.1,0.8l-0.3,2.7
        l1.8,0.6l-3.1,4.1l4.6,2.3l3.2-1.6l7.1,3.3l-5.2,4.5l-4.1-0.6l-1.4,2l-5.9-1.1l0.6,3.7l5.4-0.5l7.1,2l9.5-0.9l1-1.5l-1.1-1.5l4-3
        l3.2-1.2l5.7,0.9l0.6-4l6.4-0.8l1-2.4l6.8-3.4L1387.2,302.6L1387.2,302.6z"/>
      <path id="KH" fill={selected == 'KH' ? selectedColor : hover == 'KH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1574.8,481.8l-5.2-2.3l-2,4.3l-4.9-2.4l-5.3-1l-7.1,1.3l-3,5.2l2.1,7.7l3.4,6.6l2.6,3.3l4.7,0.9l4.7-2.5l5.8-0.5
        l-2.8-3.8l8.9-4.9l-0.1-7.7L1574.8,481.8L1574.8,481.8z"/>
      <path id="KR" fill={selected == 'KR' ? selectedColor : hover == 'KR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1637.3,331.7l6.2,5.5l-3.4,1.1l5.2,6.8l1.1,4.8l2.1,3.5l4.5-0.5l3.2-2.7l4.2-1.2l0.5-3.6l-3.4-7.5l-3.3-4.2
        l-8.2-7.6l0.1,1.6l-2.1,0.4l-3.5,0.3l-0.7,2.9l-2.4-0.2L1637.3,331.7L1637.3,331.7z"/>
      <path id="KW" fill={selected == 'KW' ? selectedColor : hover == 'KW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1235.6,381.4l-3.7-0.5l-3.2,6.1l4.9,0.6l1.7,3.1l3.8-0.2l-2.4-4.8l0.3-1.5L1235.6,381.4L1235.6,381.4z"/>
      <path id="LA" fill={selected == 'LA' ? selectedColor : hover == 'LA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1574.8,481.8l0.2-6.4l-2-4.5l-4.8-4.4l-4.3-5.6l-5.7-7.5l-7.3-3.8l1.3-2.3l3.3-1.7l-3-5.5l-6.8-0.1l-3.4-5.7
        l-4-5.1l-2.7,1l1.9,7.2l-2.9-0.1l-0.7-1.5l-4.1,4.1l-0.8,2.4l2.6,1.9l0.9,3.8l3.8,0.3l-0.4,6.7l1,5.7l5.3-3.8l1.8,1.2l3.2-0.2
        l0.8-2.2l4.3,0.4l4.9,5.2l1.3,6.3l5.2,5.5l0.5,5.4l-1.5,2.9l4.9,2.4l2-4.3L1574.8,481.8L1574.8,481.8z"/>
      <path id="LB" fill={selected == 'LB' ? selectedColor : hover == 'LB' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1167.8,360.5l0.9-3.5l2.6-2.4l-1.2-2.5l-2.4-0.3l-0.1,0.2l-2.1,4.5l-1.3,5.2h1.8l0.4-1.1L1167.8,360.5
        L1167.8,360.5z"/>
      <path id="LR" fill={selected == 'LR' ? selectedColor : hover == 'LR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M929.4,523.3l-1.6-0.2l-1.1,2.6l-1.6-0.1l-1.1-1.3l0.4-2.6l-2.3-3.9l-1.5,0.7l-1.2,0.2l-2.6,3l-2.6,3.4l-0.3,1.9
        l-1.3,2l3.7,4.1l4.8,3.5l5.1,4.8l5.7,3.1l1.5-0.1l0.5-5.2l0.5-0.8l-0.2-2.5l-2.3-2.7l-1.8-0.4l-1.6-1.8l1.2-2.8l-0.6-3.1
        L929.4,523.3L929.4,523.3z"/>
      <path id="LY" fill={selected == 'LY' ? selectedColor : hover == 'LY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1111.8,371.4l-1.5-2.1l-5.4-0.8l-1.8-1.1h-2l-2-2.8l-7.3-1.3l-3.6,0.8l-3.7,3l-1.5,3.1l1.5,4.8l-2.4,3l-2.5,1.6
        l-5.9-3.1l-7.7-2.7l-4.9-1.2l-2.8-5.7l-7.2-2.8l-4.5-1.1l-2.2,0.6l-6.4-2.2l-0.1,4.9l-2.6,1.8l-1.5,2l-3.7,2.5l0.7,2.6l-0.4,2.7
        l-2.6,1.4l1.9,5.6l0.4,3l-0.9,5.2l0.5,2.9l-0.6,3.5l0.5,4l-2.1,2.6l3.4,4.7l0.2,2.7l2,3.6l2.6-1.2l4.3,2.9l2.5,4l8.8,2.8l3.1,3.5
        l3.9-2.4l5.4-3.5l22.3,12.2l22.4,12.2v-2.7h6.3l-0.5-12.7l-1-23.4l-1.3-22.7l-2-5.1l1.2-3.9l-1.1-2.7L1111.8,371.4L1111.8,371.4z"/>
      <path id="LK" fill={selected == 'LK' ? selectedColor : hover == 'LK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1432.2,532.7l2.3-1.8l0.6-6.6l-3-6.6l-2.9-4.5l-4.1-3.5l-1.9,10.3l1.4,9.1l2.8,5.1L1432.2,532.7L1432.2,532.7z"/>
      <path id="LS" fill={selected == 'LS' ? selectedColor : hover == 'LS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1128.1,766.5l1.1-2l3.1-1l1.1-2.1l1.9-3.1l-1.7-1.9l-2.3-2l-2.6,1.3l-3.1,2.5l-3.2,4l3.7,4.9L1128.1,766.5
        L1128.1,766.5z"/>
      <path id="LT" fill={selected == 'LT' ? selectedColor : hover == 'LT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1100.4,221.2l-5-2.9l-2.5-0.4l-0.9-1.3l-4.4,0.6l-7.9-0.4l-5,1.9l1.7,5l5,1.1l2.2,0.9l-0.2,1.7l0.6,1.5l2.5,0.6
        l1.4,1.9h4.6l4.8-2.2l0.5-3.4l3.5-2L1100.4,221.2L1100.4,221.2z"/>
      <path id="LU" fill={selected == 'LU' ? selectedColor : hover == 'LU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1007,258.6l0.2-2.7l-1-1.4l-1.3,0.2l-0.4,3.5l1.1,0.5L1007,258.6z"/>
      <path id="LV" fill={selected == 'LV' ? selectedColor : hover == 'LV' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1102.1,210.1h-3.8l-4.4-2.2l-2.1-0.7l-3.7,1l-0.2,4.6l-3.6,0.1l-4.4-4.5l-4,2.1l-1.7,3.7l0.5,4.5l5-1.9l7.9,0.4
        l4.4-0.6l0.9,1.3l2.5,0.4l5,2.9l2.6-1l4.6-2.3l-2.1-3.6l-1-2.8L1102.1,210.1L1102.1,210.1z"/>
      <path id="EH" fill={selected == 'EH' ? selectedColor : hover == 'EH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M905.4,396.2 905.3,396.3 901.7,402.8 898,405.1 895.9,409.1 895.7,412.4 894.1,416.2 892.2,417.2 889.1,421.2 887.1,425.7 887.4,427.9 885.5,431.2 883.3,432.9 883,435.9 882.7,438.6 883.9,436.4 905.5,436.5 904.6,427.3 906,424 911.2,423.5 911.4,407.2 929.3,407.5 929.5,397.8 929.6,396.6 929.6,396.2 929.6,396.2 905.4,396.2"/>
      <path id="MA" fill={selected == 'MA' ? selectedColor : hover == 'MA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M969.3,363.1 967.5,356.4 967.2,352.5 965.2,348.4 962.9,348.3 957.4,346.9 952.4,347.3 949.3,344.6 945.4,344.6 
      943.6,348.5 939.9,355.2 935.9,357.8 930.5,360.7 927,365 926.1,368.4 924,373.8 925.1,381.7 920.4,387 917.7,388.7 913.3,393.1 
      908.2,393.8 905.4,396.2 929.6,396.2 929.7,388.7 938.6,384 944,383 948.4,381.3 950.5,378.1 956.8,375.6 957.1,370.9 960.2,370.4 
      962.7,368 969.7,367 970.7,364.5"/>
      <path id="MD" fill={selected == 'MD' ? selectedColor : hover == 'MD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1118.5,283.3l1.2-0.7l0.5-2.1l1.1-2l-0.5-1.1l1-0.5l0.6,0.9l3,0.2l1.2-0.5l-1-0.6l0.2-1l-2-1.5l-1.1-2.6l-1.9-1.1
        v-2.1l-2.5-1.6l-2-0.3l-3.9-1.9l-3.2,0.6l-1.1,0.9l1.6,0.6l1.8,1.9l1.9,2.6l3.4,3.7l0.6,2.7l-0.2,2.7L1118.5,283.3z"/>
      <path id="MG" fill={selected == 'MG' ? selectedColor : hover == 'MG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1255.7,658.4l-1.1-4.2l-1.4-2.7l-1.8-2.7l-2,2.8l-0.3,3.8l-3.3,4.5l-2.3-0.8l0.6,2.7l-1.8,3.2l-4.8,3.9l-3.4,3.7
        h-2.4l-2.2,1.2l-3.1,1.3l-2.8,0.2l-1,4.1l-2.2,3.5l0.1,5.9l0.8,4l1.1,3l-0.8,4.1l-2.9,4.8l-0.2,2.1l-2.6,1.1l-1.3,4.6l0.2,4.6l1.6,5
        l-0.1,5.7l1.2,3.3l4.2,2.3l3,1.7l5-2.7l4.6-1.5l3.1-7.4l2.8-8.9l4.3-12l3.3-8.8l2.7-7.4l0.8-5.4l1.6-1.5l0.7-2.7l-0.8-4.7l1.2-1.9
        l1.6,3.8l1.1-1.9l0.8-3.1l-1.3-2.9L1255.7,658.4L1255.7,658.4z"/>
      <path id="MX" fill={selected == 'MX' ? selectedColor : hover == 'MX' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M444.4,407.8l-3.6-1.4l-3.9-2l-0.8-3l-0.2-4.5l-2.4-3.6l-1-3.7l-1.6-4.4l-3.1-2.5l-4.4,0.1l-4.8,5l-4-1.9l-2.2-1.9
        l-0.4-3.5l-0.8-3.3l-2.4-2.8l-2.1-2l-1.3-2.2h-9.3l-0.8,2.6H391h-10.7l-10.7-4.4l-7.1-3.1l1-1.3l-7,0.7l-6.3,0.5l0.2,5.7l0.7,5.1
        l0.7,4.1l0.8,4l2.6,1.8l2.9,4.5l-1,2.9l-2.7,2.3l-2.1-0.3l-0.6,0.5l2.3,3.7l2.9,1.5l1,1.7l0.9-0.9l3.1,2.9l2.1,2l0.1,3.4l-1.2,4.7
        l2.5,1.6l3.3,3.1l2.9,3.6l0.7,3.9h1l2.7-2.3l0.4-1.2l-1.5-2.8l-1.6-2.9l-2.6-0.2l0.4-3.4l-0.9-3l-1-2.8l-0.5-5.9l-2.6-3.2l-0.6-2.3
        l-1.2-1.6v-4.1l-1,0.1l-0.1-2.2l-0.7-0.5l-0.4-1.4l-2.7-4.4l-1.1-2.6l1-4.8l0.1-3l1.8-2.6l2.4,1.7l1.9-0.2l3.1,2.5l-0.9,2.4l0.4,4.9
        l1.5,4.7l-0.4,2l1.7,3.1l2.3,3.4l2.7,0.5l0.3,4.4l2.4,3.1l2.5,1.5l-1.8,4l0.7,1.5l4.1,2.6l1.9,4l4.5,4.9l3.8,6.4l1.3,3.2v2.5
        l1.4,2.9l-0.3,2.2l-1.6,1.6l0.3,1.8l-1.9,0.7l0.8,3.1l2.2,4l5.3,3.6l1.9,2.9l5.4,2l3,0.4l1.2,1.7l4.2,3l5.9,3l4,0.9l4.8,2.9l4,1.2
        l3.7,1.7l2.9-0.7l4.8-2.4l3.1-0.4l4.4,1.6l2.6,2.1l5.5,6.9l0.4-1.9l0.8-1.5l-0.7-1.2l3.3-5.2h7.1l0.4-2.1l-0.8-0.4l-0.5-1.4
        l-1.9-1.5l-1.8-2.1h2.6l0.4-3.6h5.2l5.1,0.1l0.1-1l0.7-0.3l0.9,0.8l2.5-3.9h1l1.2-0.1l1.2,1.6l2-5l1.2-2.7l-0.9-1.1l1.8-3.9l3.5-3.8
        l0.6-3.1l-1.2-1.3l-3.4,0.5l-4.8-0.2l-6,1.5l-4,1.7l-1.2,1.8l-1.2,5.4l-1.8,3.7l-3.9,2.6l-3.6,1.1l-4.3,1.1l-4.3,0.6l-5.1,1.8
        l-1.9-2.6l-5.6-1.7l-1.8-3.2l-0.7-3.6l-3-4.7l-0.4-5l-1.2-3.1l-0.5-3.4l1.1-3.1l1.8-8.6l1.8-4.5l3.1-5.6L444.4,407.8L444.4,407.8z"
        />
      <path id="MK" fill={selected == 'MK' ? selectedColor : hover == 'MK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1094,304.8l-2.8-2l-2.4,0.1l-1.7,0.4l-1.1,0.2l-2.9,1l-0.1,1.2h-0.7l0,0l-0.4,2.1l0.9,2.6l2.3,1.6l3.3-0.6l1.9-1.3
        l2.8,0.1l0.7-1.1l1-0.2L1094,304.8L1094,304.8z"/>
      <path id="ML" fill={selected == 'ML' ? selectedColor : hover == 'ML' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1000.3,450.3l-6.1,0.6l-0.1-4l-2.6-1.1l-3.4-1.8l-1.3-3l-18.6-13.8l-18.4-13.9l-8.4,0.1l2.4,27.4l2.4,27.5l1,0.8
        l-1.3,4.4l-22.3,0.1l-0.9,1.4l-2.1-0.4l-3.2,1.3l-3.8-1.8l-1.8,0.2l-1,3.7l-1.9,1.2l0.2,3.9l1.1,3.7l2.1,1.8l0.4,2.4l-0.3,2l0.3,2.3
        h0.9l1.5-0.8l0.9,0.2l1.5,1.6l2.4,0.5l1.6-1.4l1.8-0.8l1.3-0.9l1.1,0.2l1.3,1.4l0.6,1.7l2.3,2.7l-1.2,1.6l-0.2,2.1l1.2-0.6l0.7,0.7
        l-0.3,1.9l1.7,1.8l0.7-0.6l1.6,1l4.3,0.1l1-1.9l1,0.1l1.6-0.7l0.9,2.7l1.3-0.8l2.3-0.9l-0.4-3.7l1.6-2.7l-0.2-2.2l4.5-5.2l0.8-4.4
        l1.6-1.6l2.7,0.9l2.3-1.3l0.8-1.6l4.3-2.9l1.1-2l5.2-2.6l3-0.9l1.4,1.2h3.6l3.6-0.3l2-2.2l7.6-0.6l4.9-1l0.5-3.9l3-4.3L1000.3,450.3
        L1000.3,450.3z"/>
      <path id="MM" fill={selected == 'MM' ? selectedColor : hover == 'MM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1533.9,435.8l-0.6-2.6l-3.8,1.8l-2.5-1.2l-4.5-2.4l0.8-5.2l-3.7-1.3l-2.3-5.8l-5.6,1l-0.7-7.5l4.1-5.3l-0.8-5.3
        l-1.3-4.9l-2.7-1.5l-2.7-3.7l-3,0.4l0.9,2.4l-1.6,1.2l1.3,3.9l-4.1-1.1l-6.1,4.4l0.9,3.6l-2.1,5.4l0.3,3.1l-1.5,5.2l-4.6-1.4
        l0.9,6.6l-0.9,2.1l0.9,2.7l-2.4,1.5l0.5,4.6l-2.1-1l1.1,5.1l4.6,5.2l3.4,0.9l-0.4,2.2l5.4,7.4l1.9,5.9l-0.9,7.9l3.6,1.5l3.2,0.6
        l5.8-4.6l3.2-3.1l3.1,5.2l2,8.1l2.6,7.6l2.6,3.3l0.2,6.9l2.2,3.8l-1.3,4.8l0.9,4.8l2.2-6.6l2.6-5.9l-2.8-5.8l-0.2-3l-1-3.5l-4.2-5.1
        l-1.7-3.2l1.7-1.1l1.4-5.6l-2.9-4.2l-4.1-4.6l-3.5-5.6l2.2-1.1l1.5-6.9l3.9-0.3l2.8-2.8l3-1.4l0.8-2.4L1533.9,435.8L1533.9,435.8z"
        />
      <path id="ME" fill={selected == 'ME' ? selectedColor : hover == 'ME' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1080,299.8l0.4-0.6l-2-1.2l-1.8-0.7l-0.8-0.8l-1.5-1.1l-0.9,0.6l-1.5,1.4l-0.4,3.4l-0.5,1l0,0l2.3,1.2l1.6,2.1
        l1.1,0.4l0,0l-0.5-1.9l2-3.1l0.4,1.2l1.3-0.5L1080,299.8z"/>
      <path id="MN" fill={selected == 'MN' ? selectedColor : hover == 'MN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1473.7,252.1l-3.7-4.6l-6.6-1.5l-4.8-0.8l-6.9-2.5l-1.3,6.4l4,3.6l-2.4,4.3l-7.9-1.6l-5-0.2l-4.7-2.9l-5.1-0.1
        l-5.3-1.9l-5.9,2.9l-6.6,5.4l-4.7,1l3.3,4.4l5.7,3.3l8.1,2.3l5.8,5l1.3,7.3l3,2.7l6.4,1l7.2,0.9l7.9,3.8l3.4,0.7l4.9,5.7l4.7,3.6
        l5.5-0.1l11.2,1.3l6.4-0.8l5.5,0.9l9.3,3.8l6.2-0.1l3.2,2l4.4-3.3l7.2-2.2l7.5-0.2l4.9-2.2l1.9-3.3l2.5-2l-1.9-2.1l-2.9-2.3l0.4-4
        l3.2,0.5l5.9,1.3l3.1-3.3l6.3-2.4l1.4-4.1l2.4-1.8l6.8-0.8l4.3,0.7l-0.7-2.2l-7.2-4.3l-5.1-2l-2.5,2.3l-5.4-1l-2.4,0.8l-2.7-2.6
        l-0.3-6.2l-0.6-4.6l-5.5,0.5l-3.9-2.1l-3.3-0.7l-4.5,4.4l-5.8,1l-3.6,1.6l-6.7-1h-4.5l-4.9-3.1l-6.5-3l-5.4-0.8l-5.7,0.8l-3.9,1.1
        L1473.7,252.1L1473.7,252.1z"/>
      <path id="MZ" fill={selected == 'MZ' ? selectedColor : hover == 'MZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1203,640.7l-0.8-2.9l0,0l0,0l-4.6,3.7l-6.2,2.5l-3.3-0.1l-2.1,1.9l-3.9,0.1l-1.4,0.8l-6.7-1.8l-2.1,0.3l-1.6,6
        l0.7,7.3h0.3l1.9,2l2.2,4.6l0.1,8.2l-2.5,1.3l-1.9,4.5l-3.4-4l-0.2-4.5l1.3-2.9l-0.3-2.6l-2.1-1.6l-1.6,0.6l-3-3l-17.1,5.2l0.3,4.5
        l0.3,2.4l4.6-0.1l2.6,1.3l1.1,1.6l2.6,0.5l2.8,2l-0.3,8.1l-1.3,4.4l-0.5,4.7l0.8,1.9l-0.8,3.7l-0.9,0.6l-1.6,4.6l-6.2,7.2l2.2,9
        l1.1,4.5l-1.4,7.1l0.4,2.3l0.6,2.9l0.3,2.8h4.1l0.7-3.3l-1.4-0.5l-0.3-2.6l2.6-2.4l6.8-3.4l4.6-2.2l2.5-2.3l0.9-2.6l-1.2-1.1l1.1-3
        l0.5-6.2l-1,0.3v-1.9l-0.8-3.7l-2.4-4.8l0.7-4.6l2.3-1.4l4.1-4.6l2.2-1.1l6.7-6.8l6.4-3.1l5.2-2.5l3.7-3.9l2.4-4.4l1.9-4.6l-0.9-3.1
        l0.2-9.9l-0.4-5.6L1203,640.7L1203,640.7z"/>
      <path id="MR" fill={selected == 'MR' ? selectedColor : hover == 'MR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M949.8,413.3l-20.3-15.5l-0.2,9.7l-17.9-0.3l-0.2,16.3L906,424l-1.4,3.3l0.9,9.2l-21.6-0.1l-1.2,2.2l2.8,2.7l1.4,3
        l-0.7,3.2l0.6,3.2l0.5,6.3l-0.8,5.9l-1.7,3.2l0.4,3.4l2-2l2.7,0.5l2.8-1.4h3.1l2.6,1.8l3.7,1.7l3.2,4.7l3.6,4.4l1.9-1.2l1-3.7
        l1.8-0.2l3.8,1.8l3.2-1.3l2.1,0.4l0.9-1.4l22.3-0.1l1.3-4.4l-1-0.8l-2.4-27.5l-2.4-27.4L949.8,413.3L949.8,413.3z"/>
      <path id="MW" fill={selected == 'MW' ? selectedColor : hover == 'MW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1169.2,661.5l0.1-2.3l-1.2-1.9l0.1-2.8l-1.5-4.7l1.7-3.5l-0.1-7.7l-1.9-4.1l0.2-0.7l0,0l-1.1-1.7l-5.4-1.2l2.6,2.8
        l1.2,5.4l-1,1.8l-1.2,5.1l0.9,5.3l-1.8,2.2l-1.9,5.9l2.9,1.7l3,3l1.6-0.6l2.1,1.6l0.3,2.6l-1.3,2.9l0.2,4.5l3.4,4l1.9-4.5l2.5-1.3
        l-0.1-8.2l-2.2-4.6l-1.9-2h-0.3v0.8l1.1,0.3l1,3.4l-0.2,0.8l-1.9-2.5l-1,1.6L1169.2,661.5L1169.2,661.5z"/>
      <path id="MY" fill={selected == 'MY' ? selectedColor : hover == 'MY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1558.1,554.4l-0.5-3.8l-0.6-2.1l0.5-2.9l-0.5-4.3l-2.6-4.3l-3.5-3.8l-1.3-0.6l-1.7,2.6l-3.7,0.8l-0.6-3.3l-4.7-2.8
        l-0.9,1.1l1.4,2.7l-0.4,4.7l2.1,3.4l1,5.3l3.4,4.3l0.8,3.2l6.7,5l5.4,4.8l4-0.5l0.1-2.1l-2.3-5.6L1558.1,554.4z M1560.9,563.3
        l0.2,0.2l-0.1,0.2l-0.9,0.4l-0.9-0.4l0.3-0.6l0.6-0.1l0.5,0.2L1560.9,563.3z M1645.2,540.2l-3.8,0.4l1.2,3.1l-4,2.1l-5-1h-6.4
        l-1.7,7.2l-2,2.2l-2.7,8.9l-4.5,1.3l-5.4-1.8l-2.7,0.6l-3.2,3.2l-3.6-0.4l-3.6,1.2l-3.9-3.5l-1-4.3l4.1,2.2l4.4-1.2l0.9-5.4l2.4-1.2
        l6.7-1.4l3.8-5l2.6-4l2.7,3.3l1.1-2.2l2.7,0.2l0.1-4.1l0.1-3.1l4.1-4.4l2.5-5h2.3l3.1,3.2l0.4,2.8l3.8,1.7l4.8,2L1645.2,540.2z"/>
      <path id="NA" fill={selected == 'NA' ? selectedColor : hover == 'NA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1105.4,683.7l-10.3,2.5l-13.4-0.9l-3.7-3l-22.5,0.3l-0.9,0.4l-3.2-2.9l-3.6-0.1l-3.3,1l-2.7,1.2l0.2,4.9l4.4,6.2
        l1.1,4l2.8,7.7l2.7,5.2l2.1,2.6l0.6,3.5v7.6l1.6,9.8l1.2,4.6l1,6.2l1.9,4.7l3.9,4.8l2.7-3.2l2.1,1.8l0.8,2.7l2.4,0.5l3.3,1.2
        l2.9-0.5l5-3.2l1.1-23.6l0.6-18.5l5.4-0.2l0.9-22.7l4.1-0.2l8.6-2.2l2,2.6l3.7-2.5h1.6l3.2-1.5V684l-2.1-1.4l-3.6-0.4L1105.4,683.7
        L1105.4,683.7z"/>
      <path id="NE" fill={selected == 'NE' ? selectedColor : hover == 'NE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1051.3,425.6l-8.8-2.8l-18.6,12.2l-15.8,12.5l-7.8,2.8l0.1,14.6l-3,4.3l-0.5,3.9l-4.9,1l-7.6,0.6l-2,2.2l-3.6,0.3
        l-0.5,3.1l0.8,2.9l3.1,4.1l0.2,3.1l6.4,1.4l-0.1,4.4l1.9-1.9h2l4.3,3.7l0.3-5.7l1.6-2.6l0.8-3.6l1.4-1.4l6-0.8l5.6,2.4l2.1,2.4
        l2.9,0.1l2.6-1.5l6.8,3.3l2.8-0.2l3.3-2.7l3.3,0.2l1.6-0.9l3,0.4l4.3,1.8l4.3-3.5l1.3,0.2l3.9,7l1-0.2l0.2-2l1.6-0.4l0.5-2.9
        l-3.6-0.2v-4.1l-2.4-2.3l2.3-8.4l6.9-6l0.2-8.3l1.8-12.9l1.1-2.7l-2.3-2.2l-0.2-2.1l-2-1.6l-1.6-9.9l-3.9,2.4L1051.3,425.6
        L1051.3,425.6z"/>
      <path id="NG" fill={selected == 'NG' ? selectedColor : hover == 'NG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1055.8,492.7l-1,0.2l-3.9-7l-1.3-0.2l-4.3,3.5l-4.3-1.8l-3-0.4l-1.6,0.9l-3.3-0.2l-3.3,2.7l-2.8,0.2l-6.8-3.3
        l-2.6,1.5l-2.9-0.1l-2.1-2.4l-5.6-2.4l-6,0.8l-1.4,1.4l-0.8,3.6l-1.6,2.6l-0.3,5.7l-0.2,2.1l1.2,3.8l-1.1,2.5l0.6,1.7l-2.7,4
        L993,514l-1,4l0.1,4.1l-0.3,10.2h4.9h4.3l3.9,4.2l1.9,4.6l3,3.9l4.5,0.2l2.2-1.4l2.1,0.3l5.8-2.3l1.4-4.5l2.7-6.1l1.6-0.1l3.3-3.7
        l2.1-0.1l3.2,2.6l3.9-2.2l0.5-2.6l1.2-2.6l0.8-3.2l3-2.6l1.1-4.5l1.2-1.4l0.7-3.3l1.5-4l4.6-5l0.3-2.1l0.6-1.1L1055.8,492.7
        L1055.8,492.7z"/>
      <path id="NI" fill={selected == 'NI' ? selectedColor : hover == 'NI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M514.1,476.8l-1.9-0.2l-0.9,0.9l-2,0.8h-1.4l-1.3,0.8l-1.1-0.3l-0.9-0.9l-0.6,0.2l-0.8,1.5l-0.5-0.1l-0.3,1.3
        l-2.1,1.8l-1.1,0.7l-0.6,0.8l-1.5-1.3l-1.4,1.7h-1.2l-1.3,0.2l-0.2,3.1h-0.8l-0.8,1.5l-1.8,0.3l-0.4,0.4l-0.9-1l-0.7,1l2.6,2.9
        l2.2,2l1,2.1l2.5,2.6l1.8,2l0.9-0.8l3.5,1.7l1.4-0.8l1.7,0.5l0.8,1.3l1.7,0.4l1.4-1.3l-0.8-1.1l-0.1-1.7l1.2-1.6l-0.2-1.7l0.7-2.7
        l0.9-0.7l0.1-2.8l-0.2-1.7l0.4-2.8l0.9-2.5l1.4-2.2l-0.3-2.3l0.4-1.4L514.1,476.8L514.1,476.8z"/>
      <path id="NL" fill={selected == 'NL' ? selectedColor : hover == 'NL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1005.5,243.9h2.9l1.1-2.3l1-5.6l-1-2l-3.9-0.2l-6.5,2.6l-3.9,8.9l-2.5,1.7l0,0l3.6,0.5l4.4-1.3l3.1,2.7l2.8,1.4
        L1005.5,243.9L1005.5,243.9z"/>
      <path id="NO" fill={selected == 'NO' ? selectedColor : hover == 'NO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1088.8,133.1l-6.9,1.1l-7.3-0.3l-5.1,4.4l-6.7-0.3l-8.5,2.3l-10.1,6.8l-6.4,4l-8.8,10.7l-7.1,7.8l-8.1,5.8
        l-11.2,4.8l-3.9,3.6l1.9,13.4l1.9,6.3l6.4,3l6-1.4l8.5-6.8l3.3,3.6l1.7-3.3l3.4-4l0.9-6.9l-3.1-2.9l-1-7.6l2.3-5.3l4.3,0.1l1.3-2.2
        l-1.8-1.9l5.7-7.9l3.4-6.1l2.2-3.9l4,0.1l0.6-3.1l7.9,0.9v-3.5l2.5-0.3l2.1-1.4l5.1,2.9l5.3-0.3l4.7,1.3l3.4-2.4l1.1-3.9l5.8-1.8
        l5.7,2.1l-0.8,3.8l3.2-0.5l6.4-2.2l0,0l-5.4-3.3l4.8-1.4L1088.8,133.1L1088.8,133.1z M1066.2,99.8l-5.6-1l-1.9-1.7l-7.2,0.9l2.6,1.5
        l-2.2,1.2l6.7,1.1L1066.2,99.8z M1040.8,91.5l-4.8-1.6l-5.1,0.2l-1,1.5h-5l-2.2-1.5l-9.3,1.6l3.2,3.5l7.6,3.8l5.7,1.4l-3,1.7
        l8.4,2.9l4.4-0.2l0.9-3.9l3-0.9l1.2-3.4l8.5-1.8C1053.3,94.8,1040.8,91.5,1040.8,91.5z M1065,88.4l-9.1-1l-3.2,1.2l-5.3-1l-10.4,1.2
        l4.3,2h5.1l0.9,1.3l10.6,0.7l10.1-0.5l4.3-2.4C1072.3,89.9,1065,88.4,1065,88.4z"/>
      <path id="NP" fill={selected == 'NP' ? selectedColor : hover == 'NP' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1455.2,394.8l-6.5-0.6l-6.4-1.5l-5-2.8l-4.5-1.2l-2.5-3.1l-3.2-0.9l-6.4-4.1l-4.7-2l-1.9,1.5l-2.8,2.9l-0.9,5.9
        l5.7,2.5l5.8,3.1l7.7,3.5l7.6,0.9l3.8,3.2l4.3,0.6l6.8,1.5l4.6-0.1l0.1-2.5l-1.5-4.1L1455.2,394.8L1455.2,394.8z"/>
      <path id="NZ" fill={selected == 'NZ' ? selectedColor : hover == 'NZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1868.6,832.8l0.9-2.6l-5.8,2.9l-3.4,3.4l-3.2,1.6l-5.9,4.6l-5.6,3.2l-7,3.2l-5.5,2.4l-4.3,1.1l-11.3,6.1l-6.4,4.6
        l-1.1,2.3l5.1,0.4l1.5,2.1l4.5,0.1l4-1.8l6.3-2.8l8.1-6.2l4.7-4.1l6.2-2.3l4-0.1l0.6-2.9l4.6-2.5l7-4.5l4.2-2.9l2.1-2.6l0.5-2.6
        l-5.6,2.5L1868.6,832.8L1868.6,832.8z M1897.4,802.3l1.9-5.7l-3.1-1.7l-0.8-3.6l-2.3,0.5l-0.4,4.6l0.8,5.7l0.9,2.7l-0.9,1.1
        l-0.6,4.4l-2.4,4.1l-4.2,5l-5.3,2.2l-1.7,2.4l3.7,2.5l-0.8,3.5l-6.9,5.1l1.4,0.9l-0.4,1.6l5.9-2.5l5.9-4.2l4.5-3.4l1.6-1.2l1.5-2.7
        l2.8-2l3.8,0.2l4.2-3.8l5.1-5.7l-2.1-0.8l-4.6,2.5l-3.2-0.5l-2.9-2.1l2.3-4.9l-1.2-1.8l-2.9,4.4L1897.4,802.3L1897.4,802.3z"/>
      <path id="OM" fill={selected == 'OM' ? selectedColor : hover == 'OM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1301,437.8l2.1-2l0.8-1.8l1.6-3.8l-0.1-1.4l-2.1-0.8l-1.6-2.1l-2.9-3.7l-3.3-1.1l-4.1-0.9l-3.3-2.3l-2.9-4.3h-2.8
        l-0.1,4.2l1.1,0.8l-2.4,1.3l0.3,2.6l-1.4,2.6l0.1,2.6l2.9,4.5l-2.6,12.7l-16.1,6.4l5.2,10.5l2.1,4.4l2.5-0.3l3.6-2.2l3.1,0.6
        l2.5-1.8l-0.2-2.5l2.1-1.6h3.4l1.2-1.3l0.2-3.1l3.3-2.4h2.6l0.4-0.8l-1-4.2l0.6-3.2l1-1.5l2.5,0.3L1301,437.8L1301,437.8z
        M1284.4,407.4l0.2-2.6l-0.7-0.6l-1.3,2.2l1.3,2.2L1284.4,407.4z"/>
      <path id="PK" fill={selected == 'PK' ? selectedColor : hover == 'PK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1388.3,346.3l-9.4-2.6l-2.9-5l-4.7-3l-2.8,0.7l-2.4,1.2l-5.8,0.8l-5.3,1.3l-2.4,2.8l1.9,2.8l1.4,3.2l-2,2.7
        l0.8,2.5l-0.9,2.3l-5.1-0.2l3,4.2l-3,1.6l-1.5,3.8l1.1,3.8l-1.7,1.8l-2.1-0.6l-4,0.9l-0.2,1.7h-4l-2.3,3.6l0.8,5.4l-6.6,2.6
        l-3.8-0.5l-0.9,1.4l-3.3-0.8l-5.3,0.9l-9.6-3.2l3.2,3.3l2.8,3.9l5.6,2.7l1,5.7l2.7,1l0.9,2.9l-7.4,3.3l-1.2,7.4l7.6-0.9l8.9-0.1
        l9.9-1.2l4.9,4.8l2.1,4.6l4.2,1.6l3.2-4.2h12l-1.8-5.5l-3.5-3.2l-1.3-4.9l-4-2.9l4.9-6.6l6.4,0.5l4.5-6.7l2.1-6.5l3.9-6.3l-1-4.5
        l3.7-3.7l-5-3.1l-2.9-4.3l-3.2-5.6l1.9-2.8l8.5,1.6l5.7-1L1388.3,346.3L1388.3,346.3z"/>
      <path id="PA" fill={selected == 'PA' ? selectedColor : hover == 'PA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M543.5,517l-2-1.8l-1.7-1.9l-2.5-1.1l-3.1-0.2l0.3-0.6l-3.1-0.4l-2,1.9l-3.5,1.3l-2.5,1.6l-2.7,0.5l-1.5-1.6
        l-0.5,0.5l-2.3-0.3l0.2-1.3l-1.9-2.3l-2.2,0.6l-0.1,2.5l1.1,1l-0.8,0.7l0.1,1.2l-0.5,1.3l-0.4,1.2l0.6,1l0.3-1.4h2.4l1.4,0.7
        l2.3,0.5l1,2.5l1.8,0.4l0.8-1.1l0.8,3.8l2.6-0.3l0.9-0.9l1.5-0.9l-2.5-3.4l0.6-1.3l1.3-0.3l2.3-1.6l1.2-2.2l2.5-0.4l2.7,1.8l1,2.1
        l1.4,0.4l-1.5,1.7l1,3.5l1.8,1.8l0.9-3.1l1.8,0.5l1.1-1.9l-1.1-3.8L543.5,517z"/>
      <path id="PE" fill={selected == 'PE' ? selectedColor : hover == 'PE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M584.3,599.5l-2.9-3.4l-1.7-0.1l3.5-6.5l-4.4-3l-3.3,0.6l-2.1-1.1l-3,1.7l-4.2-0.8l-3.4-6.7l-2.7-1.7l-1.8-3l-3.7-3
        l-1.5,0.6l0.8,4.9l-1.7,4.1l-6,6.7l-6.7,2.5l-3.3,5.5l-0.9,4.3l-3.1,2.6l-2.5-3.2l-2.3-0.7l-2.3,0.5l-0.2-2.3l1.5-1.5l-0.7-2.7
        l-4.4,4l-1.6,4.5l3,6.1l-1.7,2.8l4.1,2.6l4.5,4.1l2,4.7l2.4,2.9l6,12.7l6.2,11.7l5.4,8.4l-0.8,1.8l2.8,5.3l4.6,3.9l10.7,6.9
        l11.6,6.4l0.7,2.6l5.9,3.7l2.7-1.6l1.2-3.3l2.8-6.9l-2.8-5.3l1.1-2.1l-1.2-2.4l1.9-3.2l-0.3-5.4l-0.1-4.5l1.1-2.1l-5.5-10.3l-3,1.1
        l-2.6-0.7l-0.2-9.7l-4.4,3.8l-4.9-0.2l-2.3-3.4l-3.7-0.3l1-2.8l-3.3-3.8L562,620l1.5-1.1l-0.1-2.7l3.3-1.9l-0.7-3.4l1.3-2.2l0.4-3
        l6.2-4.3l4.6-1.2l0.7-1L584.3,599.5L584.3,599.5z"/>
      <path id="PH" fill={selected == 'PH' ? selectedColor : hover == 'PH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1684.6,518.6l-0.6-2.3l-0.8-3.2l-4.8-3l0.8,4.9l-3.9,0.2l-0.7,2.8l-4.2,1.7l-2.2-2.8l-2.8,2.4l-3.4,1.7l-1.9,5.4
        l1.1,1.9l3.9-3.6l2.7,0.3l1.5-2.7l3.8,3l-1.5,3.1l1.9,4.6l6.8,3.7l1.4-3l-2.1-4.7l2.4-3.2l2.5,6.4l1.5-5.8l-0.6-3.5L1684.6,518.6
        L1684.6,518.6z M1670.1,506.8v-6.1l-3.6,6.1l0.5-4.2l-3,0.3l-0.3,4l-1.2,1.8l-1,1.7l3.8,4.4l1.6-1.9l1.4-4L1670.1,506.8
        L1670.1,506.8z M1640,512.9l2.6-4.4l3.4-3.5l-1.5-5.2l-2.4,6.3l-2.9,4.4l-3.8,4l-2.4,4.4L1640,512.9L1640,512.9z M1657.4,496.5
        l1.2,3l-0.1,3.3l0.5,2.9l3.3-1.9l2.4-2.7l-0.2-2.6h-3.6L1657.4,496.5L1657.4,496.5z M1677.4,494.8l-1.8-2.4l-5.4-0.1l4,4.8l0.3,2.4
        l-3.3-0.5l1.2,3.9l1.7,0.3l0.7,4.5l2.5-1.4l-1.7-4l-0.4-2.1l4.5,1.7L1677.4,494.8L1677.4,494.8z M1654.5,489l-2.2-2.3l-4.8-0.2
        l3.4,4.8l2.8,3.2L1654.5,489L1654.5,489z M1648.1,454.4h-3.3l-0.9,5.8l1.1,9.9l-2.6-2l1.2,6l1.2,2.8l3.3,3.7l0.4-2.3l1.8,1.4
        l-1.5,1.7l0.1,2.6l2.9,1.4l5-0.9l4,3.8l1.1-2.4l2.5,3.4l4.8,3.1l0.2-2.9l-2-1.6l0.1-3.4l-7.5-3.6l-2.3,0.8l-3.1-0.7l-2-5.1l0.1-5.1
        l3-2.1l0.6-5.3l-2.7-4.6l0.4-2.6l-0.7-1.6l-1.5,1.6L1648.1,454.4L1648.1,454.4z"/>
      <path id="PG" fill={selected == 'PG' ? selectedColor : hover == 'PG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1850.7,615.6l0.9-1.8l-2.4-2.2l-2.5-4l-1.6-1.5l-0.5-1.9l-0.8,0.7l0.9,4.8l2.2,4l2.2,2.5L1850.7,615.6
        L1850.7,615.6z M1829.5,607l2.1-3.9l0.4-3.5l-1.1-1l-3.4,0.1l0.4,3.7l-3.3,2.3l-1.7,2.2l-3.2,0.5l-0.4-3.4l-0.8,0.1l-1,3.1l-3.1,0.5
        l-5-0.9l-0.6,1.9l3.1,1.8l4.5,1.9h2.9l3-1.5l3.2-1.6l1-1.8L1829.5,607L1829.5,607z M1801.7,619.2l-0.9-4.3l5.2-0.7l-1.1-3.3l-9.1-4
        l-0.6-3.7l-2.9-3.2l-3.7-3.3l-10.2-3.6l-9.6-4.4l-1,20.7l-1.5,20.8l5.7,0.2l3.1,1.1l4.6-2.2l-0.3-4.7l3.6-2.1l4.9-1.8l7,2.8l2.4,5.6
        l2.9,3.5l3.9,4l5.5,1l4.8,0.7l1.1,1.6l3.8-0.4l0.8-1.8l-5.6-2.7l1.8-1.2l-4.2-1.1l0.5-2.8l-3.2,0.2l-3-6.8L1801.7,619.2
        L1801.7,619.2z M1836.4,600.8l-0.5-3.3l-2-2.1l-2.1-2.6l-2.3-1.5l-1.9-1.4l-2.9-1.8l-1.6,1.5l3.9,1.9l3.1,2.7l2.4,2.1l1.2,2.4
        l0.8,3.8L1836.4,600.8L1836.4,600.8z"/>
      <path id="PL" fill={selected == 'PL' ? selectedColor : hover == 'PL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1069.4,228.3l-4.6-0.1l-0.5-1.4l-4.8-1.1l-5.7,2.1l-7.1,2.8l-3.1,1.7l1.4,3.1l-1.2,1.6l2,2.2l1.4,3.3l-0.1,2.1
        l2.3,3.9l2.4,1.9l3.7,0.6l-0.1,1.7l2.7,1.2l0.6-1.5l3.4,0.6l0.7,2l3.6,0.3l2.6,3.1l0.3,0.4l1.9-0.9l2.7,2.2l2.8-1.3l2.4,0.6l3.4-0.8
        l4.9,2.3l1.1,0.4l-1.6-2.8l3.8-5.1l2.3-0.7l0.3-1.8l-3.1-5.3l-0.5-2.7l-1.9-2.9l2.7-1.2l-0.3-2.4l-1.7-2.3l-0.6-2.7l-1.4-1.9
        l-2.5-0.6l-8.7,0.1L1069.4,228.3L1069.4,228.3z"/>
      <path id="KP" fill={selected == 'KP' ? selectedColor : hover == 'KP' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1644.7,302.3L1644.7,302.3l-5.5-3.6l0.1,3.5l-6.3,2.6l2.7,3.3l-4.6-0.2l-3.6-2l-1,4.4l-3.8,3.4l-2.1,4l3.3,1.7
        l3.4,0.7l0.8,1l0.4,3.5l1.1,1.2l-0.9,0.7l-0.1,2.9l1.9,1l1.6,0.6l0.8,1.2l1.3-0.5v-1.3l3.1,1.3l0.1-0.6l2.4,0.2l0.7-2.9l3.5-0.3
        l2.1-0.4l-0.1-1.6l-4.3-2.8l-2.6-1l0.2-0.7l-1.2-2.8l1.3-1.7l2.9-1l1-1.9l0.3-1.1l1.9-1.4l-2.8-4.5l0.3-2.1l0.9-2l2.2,0.3l0,0l0,0
        l0,0L1644.7,302.3L1644.7,302.3z"/>
      <path id="PT" fill={selected == 'PT' ? selectedColor : hover == 'PT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M937.6,335.9l-0.4-2.1l2-2.5l0.8-1.7l-1.8-1.9l1.6-4.3l-2-3.8l2.2-0.5l0.3-3l0.9-0.9l0.2-4.9l2.4-1.7l-1.3-3.1
        l-3-0.2l-0.9,0.8h-3l-1.2-3.1l-2.1,0.9l-1.9,1.6l0.1,2.1l0.9,2.2l0.1,2.7l-1.3,3.8l-0.4,2.5l-2.2,2.3l-0.6,4.2l1.2,2.4l2.3,0.6
        l0.4,4l-1,5.1l2.8-0.7l2.7,0.9L937.6,335.9L937.6,335.9z"/>
      <path id="PY" fill={selected == 'PY' ? selectedColor : hover == 'PY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M655.7,700.5l-0.3-1.9l-5.4-3.3l-5.1-0.1l-9.5,1.9l-2.1,5.6l0.2,3.4l-1.5,7.6l11.2,10.4l4.6,1l7.2,4.7l5.9,2.5
        l1.1,2.8l-4.2,9.6l5.7,1.8l6.2,1l4.2-1.1l4.3-4.8l0.3-5.7l0.7-3.6l0.3-3.8l-0.3-3.5l-2.1-1.2l-2,1.1l-2-0.3l-0.9-2.5l-1-5.8
        l-1.2-1.9l-3.9-1.7l-2.1,1.2l-6-1.2l-0.4-8.6L655.7,700.5L655.7,700.5z"/>
      <path id="PS" fill={selected == 'PS' ? selectedColor : hover == 'PS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1166.9,366.1l-2-0.9l-0.7,4.3l1.4,0.7l-1.2,0.8l-0.1,1.7l2.4-0.8l0.6-1.9L1166.9,366.1L1166.9,366.1z"/>
      <path id="QA" fill={selected == 'QA' ? selectedColor : hover == 'QA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1258,415.5l0.8-3.8l-0.5-3.7l-1.9-2l-1.4,0.7l-1.1,3.3l0.8,4.7l1.8,1.2L1258,415.5L1258,415.5z"/>
      <path id="RO" fill={selected == 'RO' ? selectedColor : hover == 'RO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1108.1,266.3h-2.1l-1,1.5l-3.6,0.6l-1.6,0.9l-2.4-1.5h-3.2l-3.2-0.7l-1.9,1.3l-2.9,1.3l-1.9,4.2l-2.6,4.3l-3.8,1.1
        l2.9,2.5l0.8,1.9l3.2,1.5l0.7,2.5l3.1,1.8l1.4-1.3l1.4,0.7l-1.1,1.1l1,1l1.8,2.6l1.9-0.5l4,1l7.5,0.3l2.3-1.6l5.8-1.4l4,2.2l3,0.7
        l0.4-7.4l1.6,0.5l2.3-1.3l-0.4-1.6l-2.4-1.1l-2.2,1l-2.4-1.1l-1.3-2.8l0.2-2.7l-0.6-2.7l-3.4-3.7l-1.9-2.6l-1.8-1.9L1108.1,266.3
        L1108.1,266.3z"/>
      <path id="RU" fill={selected == 'RU' ? selectedColor : hover == 'RU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1332.3,95.1l-4.5-4l-13.6-4.1l-9.4-2.1l-6.2,0.9l-5.3,2.9l5.8,0.8l6.6,3.2l8,1.7l11.5,1.3
        C1325.2,95.7,1332.3,95.1,1332.3,95.1z M1153.6,87.8l0.9-0.6l-5.7-0.9L1146,87l-1.3,1l-1.5-1.2l-5.2,0.1l-6.2,0.8l7.7,0.1l-1.1,1.3
        l4.4,1l3.6-0.7l0.1-0.7l2.9-0.3C1149.4,88.4,1153.6,87.8,1153.6,87.8z M1354.1,97.7l-1.5-1.8l-12.5-2.6l-3-0.3l-2.2,0.5l1.2,6
        C1336.1,99.5,1354.1,97.7,1354.1,97.7z M1369.3,104l-9.2-0.7l3.4-1.2l-8.2-1.5l-6.1,1.9l-1,2l1.5,2.1l-6.9-0.1l-5.3,2.6l-4.3-1.1
        l-9.3,0.5l0.3,1.3l-9.2,0.7l-4.9,2.4l-4.2,0.2l-1.2,3.3l5.5,2.6l-7.7,0.7l-9.5-0.3l-5.8,1.1l4.8,5.4l6.9,4.3l-9.6-3l-7.9,0.3l-5.1,2
        l4.5,3.8l-4.9-1l-2.1-5l-4.2-2.8l-1.8,0.1l3.6,3.7l-4.6,3.5l8.1,4.2l0.4,5.4l2.9,2.9l4.7,0.5l0.4,3.5l4.4,3.1l-1.9,2.6l0.5,2.7
        l-3.7,1.4l-0.5,2l-5.3-0.8l3.5-7.8l-0.5-3.6l-6.7-3.3l-3.8-7.3l-3.7-3.7l-3.6-1.6l0.8-4.2l-2.9-2.9l-11.3-1.4l-2.1,1l0.5,4.7
        l-4.3,4.7l1.2,1.7l4.7,4.1l0.1,2.6l5.3,0.5l0.8,1.1l5.8,2.9l-1,2.8l-18.5-6.1l-6.6-1.7l-12.8-1.6l-1.2,1.7l5.9,3.1l-2.7,3.6
        l-6.4-3.2l-5,2.2l-7.6,0.1l-2.1,1.9l-5.3-0.6l2.5-3.3l-3.2-0.2l-12.3,4.6l-7.6,2.6l0.4,3.5l-6,1.2l-4-1.9l-1.2-3l5-0.7l-3.6-3
        l-12.2-1.8l4.3,3.4l-0.8,3.2l4.7,3.3l-1.1,3.8l-4.6-1.9l-4-0.3l-8,5.4l4.2,4.1l-3.2,1.4l-11.4-3.5l-2.1,2.1l3.3,2.4l0.2,2.7
        l-3.8-1.4l-6-1.7l-1.9-5.8l-1-2.6l-8-4l2.9-0.7l20.1,4.2l6.4-1.5l3.7-2.9l-1.6-3.6l-4-2.6l-17.6-6.1l-11.6-1.3l-7.6-3.2l-3.6,1.8
        l0,0l-6.4,2.2l-3.2,0.5l0.4,3.7l7.2,3.7l-2.8,4.1l6.4,6.3l-1.7,4.8l4.9,4.1l-0.9,3.7l7.3,3.9l-0.9,2.9l-3.3,3.3l-7.9,7.4l0,0
        l5.3,2.8l-4.5,3.2l0,0l0.9,1l-2.6,3.4l2.5,5.5l-1.6,1.9l2.4,1.4l1,2.8l2.1,3.6l5.2,1.5l1,1.4l2.3-0.7l4.8,1.4l1,2.9l-0.6,1.6
        l3.7,3.9l2.2,1.1l-0.1,1.1l3.4,1.1l1.7,1.6l-1.6,1.3l-3.9-0.2l-0.8,0.6l1.5,2l2,3.9l0,0l1.8,0.2l1-1.4l1.5,0.3l4.8-0.5l3.8,3.4
        l-0.9,1.3l0.7,1.9l4,0.2l2.2,2.7l0.2,1.2l6.6,2.2l3.5-1l3.6,2.9l2.9-0.1l7.6,2l0.4,1.9l-1.3,3.2l1.8,3.4l-0.3,2.1l-4.7,0.5l-2.2,1.7
        l0.4,2.8l4.2-1l0.4,1.3l-6.8,2.6l3.2,2.4l-3.2,5.2l-3.4,1l5,3.6l6.2,2.4l7.4,5.1l0.5-0.7l4.5,1.1l7.7,1l7.5,2.9l1.1,1.2l2.9-1
        l5.1,1.3l2.1,2.5l3.5,1.4l1.5,0.2l4.3,3.8l2.4,0.4l0.5-1.5l2.6-2.5l0,0l-7.3-7.3l-0.4-4.1l-5.9-5.9l3.5-6.3l4.6-1.1l1.4-3.7l-2.8-1
        l-0.2-3.2l-4.2-4.1l-3.6,0.2l-5.3-4.3l1.7-4.7l-1.7-1.2l2.1-6.8l6,3.6l-0.7-4.6l8.1-6.6l7.5-0.2l11.9,4.3l6.6,2.4l4.3-2.5l7.6-0.2
        l7.3,3.2l0.8-1.8l6.9,0.3l0.2-3l-9.4-4.2l3.6-2.9l-1.5-1.7l3.9-1.6l-5-4.1l1.4-2.1l16.8-2.1l1.7-1.5l10.8-2.2l3.1-2.5l9,1.3l4.3,6.3
        l4.3-1.5l7,2.1l1.2,3.3l4.4-0.4l9.1-5.7l-0.8,1.9l8.3,4.7l18.1,15.5l1.1-3.3l8.3,3.6l6.2-1.6l3.2,1.1l4.1,3.6l3.9,1.2l3.3,2.6l6-0.9
        l4.3,3.8l1.7-0.5l4.7-1l6.6-5.4l5.9-2.9l5.3,1.9l5.1,0.1l4.7,2.9l5,0.2l7.9,1.6l2.4-4.3l-4-3.6l1.3-6.4l6.9,2.5l4.8,0.8l6.6,1.5
        l3.7,4.6l8.4,2.6l3.9-1.1l5.7-0.8l5.4,0.8l6.5,3l4.9,3.1h4.5l6.7,1l3.6-1.6l5.8-1l4.5-4.4l3.3,0.7l3.9,2.1l5.5-0.5l7.3,2.3l4.4-3.9
        l-1.9-2.7l-0.1-6.5l1.2-2l-2.5-3.3l-3.7-1.5l1.7-3l5.1-1.1l6.2-0.2l8.5,1.8l5.9,2.3l7.7,6.1l3.8,2.7l4.4,3.7l6.1,6.1l9.9,1.9
        l8.9,4.5l6,5.8h7.5l2.6-2.5l6.9-1.8l1.3,5.6l-0.4,2.3l2.8,6.8l0.6,6l-6.8-1.1l-2.9,2.2l4.7,5.3l3.8,7.3l-2.5,0.1l1.9,3.1l0,0
        l1.4,1.1l0,0l0,0l0,0l-0.4-2l4-4.5l5.1,3l3.2-0.1l4.4-3.6l1-3.7l2.1-7.1l1.9-7.2l-1.3-4.3l1-9l-5.2-9.9l-5.5-7.3l-1.3-6.2l-4.7-5.1
        l-12.7-6.7l-5.6-0.4l-0.3,3l-5.8-1.3l-5.7-3.8l-8-0.7l4.9-14.1l3.5-11.5l13.1-1.8l14.9,1l2.5-2.8l7.9,0.8l4.3,4.3l6.4-0.6l8.4-1.6
        l-7.7-3.5v-9.8l9.1-1.9l12.1,7.1l3.6-6.4l-3.2-4.7l4.7-0.5l6.5,8.1l-2.4,4.6l-0.8,6l0.3,7.5l-5.7,1.3l2.8,2.7l-0.1,3.6l6.4,8.3
        l16,13.4l10.5,8.8l5.7,4.3l1.6-5.7l-4.5-6.2l5.7-1.5l-5.4-6.9l5-3.1l-4.7-2.6l-3.4-5l4.1-0.2l-9-8.6l-6.7-1.4l-2.9-2.4l-1.1-5.6
        l-3.1-3.9l7,0.8l1.3-2.5l4.7,2.2l6.1-4.6l11.4,4l-1.7-2.6l2-3.6l1.5-4l3.1-0.7l6.5-4.3l9.8,1.2l-0.9-1.5l-3.8-2.3l-4.1-1.6l-9.1-4.6
        l-8.1-3l6.1,0.4l2-2.5l0,0l-32.9-21.9l-9.4-2.3l-15.7-2.6l-7.9,0.3l-15.2-1.4l1.8,2.3l8.5,3.4l-2.5,1.8l-14.2-4.8l-6.8,0.6l-9.2-1.1
        l-7,0.2l-3.9,1.1l-7.2-1.6l-5.1-3.8l-6.5-2.2l-9.2-0.9l-14.7,1l-16.1-4l-7.8-3l-40.1-3.4l-2.1,2.2l9.3,4.8l-7.5-0.7l-1,1.5l-9.7-1.6
        l-5,1.4l-9.3-2.4l3,5.5l-8.9-2.1l-10-4.1l-0.4-2.2l-6-3.3l-9.8-2.6h-6.1l-9.3-0.9l4.7,3.9l-17.2-0.8l-3.9-2.3l-13.3-0.9l-5.3,0.8
        l-0.1,1.3l-5.8-3.2l-2.3,0.9l-7.2-1.2l-5.6-0.7l1.1-1.5l6.6-2.8l2.3-1.5l-2.4-2.5l-5.5-1.9l-11.5-2.3l-10.8-0.1l-1.9,1.2L1369.3,104
        L1369.3,104z M1207.1,135.6l-9.9-4.3l-3.1-4.3l3.3-4.9l2.8-5l8.6-4.7l9.8-2.4l11.3-2.4l1.3-1.5l-4.2-1.9l-6.6,0.6l-4.9,1.8
        l-11.7,0.9l-10.1,3.1l-6.8,2.7l2.5,2.2l-6.6,4.4l3.9,0.7l-5.4,4.3l1.6,2.8l-3.4,1.1l1.9,2.8l7.9,1.4l2.2,2.3l13.4,0.7L1207.1,135.6
        L1207.1,135.6z M1521.1,110.9l-17.9-2.6l-10.2-0.2l-3.4,0.9l3.4,3.4l12.4,3.2l4.5-1.2l14.2,0.2
        C1524.1,114.6,1521.1,110.9,1521.1,110.9z M1546.3,113.2l-11.7-1.3l-8.2-0.7l1.7,1.6l10.3,2l6.8,0.4L1546.3,113.2L1546.3,113.2z
        M1533.8,122.7l-2.5-1.4l-8.3-1.9l-4.1,0.5l-0.8,2l1.1,0.2l8.8,0.6C1528,122.7,1533.8,122.7,1533.8,122.7z M1696.4,135l-6-3.6
        l-1.4,2.2l3.5,1.6L1696.4,135z M1084,228.9l-0.6-1.5l0.2-1.7l-2.2-0.9l-5-1.1l-6.3,2l-0.7,2.6l5.9,0.7L1084,228.9z M1673.7,250.7
        l-7.2-6.2l-5.1-6l-6.8-5.8l-4.9-4l-1.3,0.8l4.4,2.8l-1.9,2.8l6.8,8.3l7.8,6l6.4,8.3l2.4,4.6l5.5,6.8l3.8,6l4.6,5.2l-0.1-4.8l6.5,3.8
        l-3-4.4l-9.5-6.3l-3.7-9l8.9,2L1673.7,250.7L1673.7,250.7z"/>
      <path id="RW" fill={selected == 'RW' ? selectedColor : hover == 'RW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1147.6,579.4l-3.3,1.9l-1.4-0.6l-1.6,1.8l-0.2,3.8l-0.8,0.4l-0.6,3.5l3.5,0.5l1.7-3.6l3,0.4l0,0l1.6-0.8l0.4-3.7
        L1147.6,579.4L1147.6,579.4z"/>
      <path id="SA" fill={selected == 'SA' ? selectedColor : hover == 'SA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1228.7,387l-10.2-0.5l-16.7-12.7l-8.5-4.5l-6.7-1.7l-0.9,1l-10.4,3.1l6.1,6.4l-1.7,1l-0.7,2.2l-4,0.8l-1.1,2.3
        l-2.1,2l-6.1-1l-0.5,2.5v2.2l-0.6,3.5h2.7l3.2,4.4l3.7,5.1l2.5,4.7l1.7,1.5l1.7,3.3l-0.2,1.4l2.1,3.7l3,1.3l2.8,2.5l3.6,7v3.8
        l0.9,4.4l4,6.1l2.5,1l4.1,4.4l1.9,5.2l3.2,5.3l3,2.3l0.6,2.5l1.8,1.9l0.9,2.8l2.3-2.1l-0.7-2.7l1.2-3.1l2.4,1.7l1.5-0.6l6.4-0.2
        l1,0.7l5.4,0.6l2.1-0.3l1.6,2.1l2.5-1l3.5-6.7l5-2.9l15.7-2.4l16.1-6.4l2.6-12.7l-2.9-4.5l-1,1.3l-16.8-3.2l-2.6-6.4l-0.4-1.5
        l-1.2-2.4l-1.5,0.4l-1.8-1.2l-1-1.6l-0.9-2.1l-1.7-1.8l-1-2.1l0.4-2.1l-0.6-2.7l-4-2.6l-1.2-2.3l-2.9-1.4l-2.7-5.5l-3.8,0.2
        l-1.7-3.1L1228.7,387L1228.7,387z"/>
      <path id="SD" fill={selected == 'SD' ? selectedColor : hover == 'SD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1180.8,468.5l0.4-4.2l1.6-2l4-1l2.6-3.6l-3.1-2.4l-2.2-1.6l-2.5-7.6l-1.1-6.5l1.1-1.2l-2.1-6.2h-21.8h-21.4h-22.1
        l0.5,12.7h-6.3v2.7l1.1,25.2l-4.8-0.4l-2.4,4.7l-1.4,3.9l1.2,1.5l-1.8,1.9l0.7,2.7l-1.4,2.6l-0.5,2.4l2-0.4l1.2,2.5l0.1,3.7l2.1,1.8
        v1.6l0.7,2.7l3.3,4v2.6l-0.8,2.6l0.3,2l2,1.8l0.5,0.3l1.7-0.7l1.9-1.2l1.3-5.7l1.5-2.9l4-0.9l1,1.8l3,3.7l1.5,0.5l2-1.1l4.1,0.3
        l0.8,1.3h5.5l0.2-1.3l2.9-1.2l0.5-1.9l2.1-1.3l4.8,3.7l2.8-0.7l2.7-4.5l3-3.5l-0.6-3.9l-1.4-1.8l3.4-0.3l0.3-1.5l2.6,0.5l-0.5,4.7
        l0.8,4.6l2.9,2.5l0.7,2.2v3.1l0.8,0.1v-0.7l1.4-6.7l2.6-1.8l0.5-2.6l2.3-4.8l3.2-3.2l2.1-6.2l0.7-5.5l-0.7-2.5L1180.8,468.5
        L1180.8,468.5z"/>
      <path id="SS" fill={selected == 'SS' ? selectedColor : hover == 'SS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1166,508.7l-0.7-2.2l-2.9-2.5l-0.8-4.6l0.5-4.7l-2.6-0.5l-0.3,1.5l-3.4,0.3l1.4,1.8l0.6,3.9l-3,3.5l-2.7,4.5
        l-2.8,0.7l-4.8-3.7l-2.1,1.3l-0.5,1.9l-2.9,1.2l-0.2,1.3h-5.5l-0.8-1.3l-4.1-0.3l-2,1.1l-1.5-0.5l-3-3.7l-1-1.8l-4,0.9l-1.5,2.9
        l-1.3,5.7l-1.9,1.2l-1.7,0.7l3.8,2.5l3.1,2.6l0.1,2l3.8,3.4l2.4,2.7l1.5,3.8l4.2,2.5l0.9,2.1l3.5,5.2l2.5,0.8l1.5-1.1l2.6,0.4
        l3.1-1.3l1.4,2.7l5,4.2l0,0l2.3-1.7l3.5,1.4l4.5-1.5l4,0.1l3.4-3l3.4-3.8l3.8-4.2l-3.5-6.9l-2.6-1.5l-1-2.5l-2.9-3.1l-3.4-0.5
        l1.8-3.6l3-0.1l0.8-2l-0.2-5l-0.8-0.1L1166,508.7L1166,508.7z"/>
      <path id="SN" fill={selected == 'SN' ? selectedColor : hover == 'SN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M908.9,479.2l-3.6-4.4l-3.2-4.7l-3.7-1.7l-2.6-1.8h-3.1l-2.8,1.4l-2.7-0.5l-2,2l-1.3,3.3l-2.8,4.4l-2.5,1.2l2.7,2.3
        l2.2,5l6.1-0.2l1.3-1.5l1.8-0.1l2.1,1.5l1.8,0.1l1.8-1.1l1.1,1.8l-2.4,1.5l-2.4-0.1l-2.4-1.4l-2.1,1.5h-1l-1.4,0.9l-5-0.1l0.8,4.9
        l3-1.1l1.8,0.2l1.5-0.7l10.3,0.3l2.7,0.1l4,1.5l1.3-0.1l0.4-0.7l3,0.5l0.8-0.4l0.3-2l-0.4-2.4l-2.1-1.8l-1.1-3.7L908.9,479.2
        L908.9,479.2z"/>
      <path id="SL" fill={selected == 'SL' ? selectedColor : hover == 'SL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M919.4,518.7l-1.5,0.3v-2.3L917,515l0.2-1.8l-1.2-2.7l-1.5-2.3H910l-1.3,1.2l-1.5,0.2l-1,1.4l-0.7,1.7l-3,2.8
        l0.7,4.7l0.9,2.3l2.9,3.5l4.1,2.5l1.5,0.5l1.3-2l0.3-1.9l2.6-3.4L919.4,518.7L919.4,518.7z"/>
      <path id="SV" fill={selected == 'SV' ? selectedColor : hover == 'SV' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M487.2,487l0.6-2.5l-0.7-0.7l-1.1-0.5l-2.5,0.8l-0.1-0.9l-1.6-1l-1.1-1.3l-1.5-0.5l-1.4,0.4l0.2,0.7l-1.1,0.7
        l-2.1,1.6l-0.2,1l1.4,1.3l3.1,0.4l2.2,1.3l1.9,0.6l3.3,0.1L487.2,487L487.2,487z"/>
      <path id="RS" fill={selected == 'RS' ? selectedColor : hover == 'RS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1084.8,285.2l-3.2-1.5l-0.8-1.9l-2.9-2.5l-3.2-0.2l-3.7,1.6l0,0l1.5,2.4l1.7,1.8l-1.7,2.3l0,0h1.8l-1,2.7l2.7,2.3
        l-0.5,2.9l-1.2,0.3l1.5,1.1l0.8,0.8l1.8,0.7l2,1.2l-0.4,0.6l1.2-0.5l0.5-2l0.9-0.4l0.8,0.9l1,0.4l0.8,1l0.8,0.3l1.1,1.1h0.8
        l-0.5,1.5l-0.5,0.7l0.2,0.5l1.7-0.4l2.4-0.1l0.7-0.9l-0.6-0.7l0.7-2l1.7-1.9l-2.8-2.6l-0.7-2.3l1.1-1.4l-1-1l1.1-1.1l-1.4-0.7
        l-1.4,1.3l-3.1-1.8L1084.8,285.2L1084.8,285.2z"/>
      <path id="SR" fill={selected == 'SR' ? selectedColor : hover == 'SR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M668,533.8l-4.6,0.5l-0.6,1.1l-6.7-1.2l-1,5.7l-3.5,1.6l0.3,1.5l-1.1,3.3l2.4,4.6l1.8,0.1l0.7,3.5l3.3,5.6l3.1,0.5
        l0.5-1.3l-0.9-1.3l0.5-1.8l2.3,0.6l2.7-0.7l3.2,1.4l1.4-2.7l0.6-2.9l1-2.8l-2.1-3.7l-0.4-4.4l3.1-5.5L668,533.8L668,533.8z"/>
      <path id="SK" fill={selected == 'SK' ? selectedColor : hover == 'SK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1087.4,260.9l-4.9-2.3l-3.4,0.8l-2.4-0.6l-2.8,1.3l-2.7-2.2l-1.9,0.9l-0.3-0.4h-1.5l-0.6,1.1l-1.1,0.3l-0.2,1.4
        l-0.9,0.3l-0.1,0.6l-1.6,0.6l-2.2-0.1l-0.6,1.4l-0.3,0.8l0.7,2.1l2.6,1.6l1.9,0.7l4.1-0.8l0.3-1.2l1.9-0.2l2.3-1l0.6,0.4l2.2-0.7
        l1-1.5l1.6-0.4l5.5,1.9l1-0.6l0.7-2.5L1087.4,260.9L1087.4,260.9z"/>
      <path id="SI" fill={selected == 'SI' ? selectedColor : hover == 'SI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1059.4,277l-1.2-2.1l-0.8-0.1l-0.9,1.1l-4.3,0.1l-2.4,1.4l-4.2-0.4l-0.3,3l1.4,2.7l-1.1,0.5l3.5,0.2l0.8-1l1.8,1
        l2,0.1l-0.2-1.7l1.7-0.6l0.3-2.5L1059.4,277L1059.4,277z"/>
      <path id="SE" fill={selected == 'SE' ? selectedColor : hover == 'SE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1077.7,161.1l-1.9-2.2l-1.7-8.4l-7.2-3.7l-5.9-2.7l-2.5,0.3v3.5l-7.9-0.9l-0.6,3.1l-4-0.1l-2.2,3.9l-3.4,6.1
        l-5.7,7.9l1.8,1.9l-1.3,2.2l-4.3-0.1l-2.3,5.3l1,7.6l3.1,2.9l-0.9,6.9l-3.4,4l-1.7,3.3l4.2,8.4l4.4,6.7l2,5.7l5.3-0.3l2.2-4.7
        l5.7,0.5l2-5.5l0.6-10l4.6-1.3l3.3-6.6l-4.8-3.3l-3.6-4l2.1-8.1l7.7-4.9l6.1-4.5l-1.2-3.5l3.4-3.9L1077.7,161.1L1077.7,161.1z"/>
      <path id="SZ" fill={selected == 'SZ' ? selectedColor : hover == 'SZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1150.5,736.6l-2.7-1.2l-1.6,0.5l-0.7,1.8l-1.6,2.4l-0.1,2.2l3,3.5l3.3-0.7l1.3-2.8l-0.3-2.8L1150.5,736.6
        L1150.5,736.6z"/>
      <path id="SY" fill={selected == 'SY' ? selectedColor : hover == 'SY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1183.5,359.9l11-6.7l0.9-7.8l-1.2-4.7l2.7-1.6l2.1-4.1l-5.9,1.1l-2.8-0.2l-5.7,2.5h-4.3l-3-1.2l-5.5,1.8l-1.9-1.3
        l0.1,3.6l-1.2,1.5l-1.2,1.4l-1,2.6l1.1,5l2.4,0.3l1.2,2.5l-2.6,2.4l-0.9,3.5l0.3,2.6l-0.6,1h0.1l6.3,2.5L1183.5,359.9L1183.5,359.9z
        "/>
      <path id="TD" fill={selected == 'TD' ? selectedColor : hover == 'TD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1108.4,447.6l-22.4-12.2l-22.3-12.2l-5.4,3.5l1.6,9.9l2,1.6l0.2,2.1l2.3,2.2l-1.1,2.7l-1.8,12.9l-0.2,8.3l-6.9,6
        l-2.3,8.4l2.4,2.3v4.1l3.6,0.2l-0.5,2.9l2.2,4.1l0.5,4.2l-0.2,4.3l3.1,5.8l-3.1-0.1l-1.6,0.4l-2.5-0.6l-1.2,3l3.3,3.7l2.5,1.1
        l0.8,2.6l1.8,4.4l-0.9,1.7l4.7-0.4l1-1.7l0.9,0.2l1.4,1.4l7.1-2.4l2.3-2.5l2.9-2.2l-0.6-2.3l1.6-0.6l5.4,0.4l5.2-2.9l4-7l2.8-2.5
        l3.5-1.1v-1.6l-2.1-1.8l-0.1-3.7l-1.2-2.5l-2,0.4l0.5-2.4l1.4-2.6l-0.7-2.7l1.8-1.9l-1.2-1.5l1.4-3.9l2.4-4.7l4.8,0.4L1108.4,447.6
        L1108.4,447.6z"/>
      <path id="TG" fill={selected == 'TG' ? selectedColor : hover == 'TG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M981.7,502.2l-4.9-0.1l-0.4,1.9l2.4,3.3l-0.1,4.6l0.6,5.1l1.4,2.3l-1.2,5.7l0.4,3.2l1.5,4l1.2,2.2l4.6-1.3l-1.4-4.4
        l0.2-14.6l-1.1-1.3l-0.2-3.1l-2-2.3l-1.7-1.9L981.7,502.2L981.7,502.2z"/>
      <path id="TH" fill={selected == 'TH' ? selectedColor : hover == 'TH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1562.7,481.4l1.5-2.9l-0.5-5.4l-5.2-5.5l-1.3-6.3l-4.9-5.2l-4.3-0.4l-0.8,2.2l-3.2,0.2l-1.8-1.2l-5.3,3.8l-1-5.7
        l0.4-6.7l-3.8-0.3l-0.9-3.8l-2.6-1.9l-3,1.4l-2.8,2.8l-3.9,0.3l-1.5,6.9l-2.2,1.1l3.5,5.6l4.1,4.6l2.9,4.2l-1.4,5.6l-1.7,1.1
        l1.7,3.2l4.2,5.1l1,3.5l0.2,3l2.8,5.8l-2.6,5.9l-2.2,6.6l-1.3,6.1l-0.3,3.9l1.2,3.6l0.7-3.8l2.9,3.1l3.2,3.5l1.1,3.2l2.4,2.4
        l0.9-1.1l4.7,2.8l0.6,3.3l3.7-0.8l1.7-2.6l-3.1-3.3l-3.4-0.8l-3.3-3.6l-1.4-5.5l-2.6-5.8l-3.7-0.2l-0.7-4.6l1.4-5.6l2.2-9.3l-0.2-7
        l4.9-0.1l-0.3,5l4.7-0.1l5.3,2.9l-2.1-7.7l3-5.2l7.1-1.3L1562.7,481.4L1562.7,481.4z"/>
      <path id="TJ" fill={selected == 'TJ' ? selectedColor : hover == 'TJ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1344.1,315.7l-2.1,0.2l-1.3-1.8l0.2-2.9l-6.4,1.5l-0.5,4l-1.5,3.5l-4.4-0.3l-0.6,2.8l4.2,1.6l2.4,4.7l-1.3,6.6
        l1.8,0.8l3.3-2.1l2.1,1.3l0.9-3l3.2,0.1l0.6-0.9l-0.2-2.6l1.7-2.3l3.2,1.5v2l1.6,0.3l1,5.4l2.6,2.1l1.5-1.3l2.1-0.7l2.5-2.9l3.8,0.5
        h5.4l-1.8-3.7l-0.6-2.5l-3.5-1.4l-1.6,0.6l-3-5.9l-9.5,0.9l-7.1-2l-5.4,0.5l-0.6-3.7l5.9,1.1L1344.1,315.7L1344.1,315.7z"/>
      <path id="TM" fill={selected == 'TM' ? selectedColor : hover == 'TM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1325.6,334.2l-0.8-4l-7.7-2.7l-6.2-3.2l-4.2-3l-7-4.4l-4.3-6.4l-2-1.2l-5.5,0.3l-2.3-1.3l-1.9-4.9l-7.8-3.3
        l-3.3,3.6l-3.8,2.2l1.6,3.1l-5.8,0.1l-2.5,0.3l-4.9-4.9l-3.8-1.7l-5.5,1.3l-1.8,2l2.5,4l-0.5-4.5l3.7-1.6l2.4,3.6l4.6,3.7l-4,2
        l-5.3-1.5l0.1,5.2l3.5,0.4l-0.4,4.4l4.5,2.1l0.7,6.8l1.8,4.5l4.4-1.2l3-3.7l3.5,0.2l2.1-1.2l3.8,0.6l6.5,3.3l4.3,0.7l7.3,5.7
        l3.9,0.2l1.6,5.5l5.9,2.4l3.9-0.8l0.4-3l4-0.9l2.5-2l-0.1-5.2l4.1-1.2l0.3-2.3l2.9,1.7L1325.6,334.2L1325.6,334.2z"/>
      <path id="TL" fill={selected == 'TL' ? selectedColor : hover == 'TL' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1676.8,631.9l4.9-1.8l6-2.8l2.2-1.7l-2-0.8l-1.8,0.8l-4,0.2l-4.9,1.4l-0.8,1.5l0.5,1.3L1676.8,631.9L1676.8,631.9z
        "/>
      <path id="TN" fill={selected == 'TN' ? selectedColor : hover == 'TN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1038,361.4l-2-1l-1.5-3l-2.8-0.1l-1.1-3.5l3.4-3.2l0.5-5.6l-1.9-1.6l-0.1-3l2.5-3.2l-0.4-1.3l-4.4,2.4l0.1-3.3
        l-3.7-0.7l-5.6,2.6l-1,3.3l1,6.2l-1.1,5.3l-3.2,3.6l0.6,4.8l4.5,3.8v1.5l3.4,2.6l2.6,11.3l2.6-1.4l0.4-2.7l-0.7-2.6l3.7-2.5l1.5-2
        l2.6-1.8L1038,361.4L1038,361.4z"/>
      <path id="TR" fill={selected == 'TR' ? selectedColor : hover == 'TR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1166.6,308.9l-9.7-4.4l-8.5,0.2l-5.7,1.7l-5.6,4l-9.9-0.8l-1.6,4.8l-7.9,0.2l-5.1,6.1l3.6,3l-2,5l4.2,3.6l3.7,6.4
        l5.8-0.1l5.4,3.5l3.6-0.8l0.9-2.7l5.7,0.2l4.6,3.5l8-0.7l3.1-3.7l4.6,1.5l3.2-0.6l-1.7,2.4l2.3,3l1.2-1.4l1.2-1.5l-0.1-3.6l1.9,1.3
        l5.5-1.8l3,1.2h4.3l5.7-2.5l2.8,0.2l5.9-1.1l2.1-1l6.2,0.9l2.1,1.6l2.3-1.1l0,0l-3.7-5.2l0.7-2l-2.9-7.3l3.3-1.8l-2.4-1.9l-4.2-1.5
        v-3.1l-1.3-2.2l-5.6-3l-5.4,0.3l-5.5,3.2l-4.5-0.6l-5.8,1L1166.6,308.9L1166.6,308.9z M1117,312.9l2-1.9l6.1-0.4l0.7-1.5l-4.7-2
        l-0.9-2.4l-4.5-0.8l-5,2l2.7,1.6l-1.2,3.9l-1.1,0.7l0.1,1.3l1.9,2.9L1117,312.9L1117,312.9z"/>
      <path id="TW" fill={selected == 'TW' ? selectedColor : hover == 'TW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1642.3,427.2l1.2-10.2l0.1-3.9l-2.9-1.9l-3.3,4.8l-1.9,6.3l1.5,4.7l4,5.4L1642.3,427.2L1642.3,427.2z"/>
      <path id="TZ" fill={selected == 'TZ' ? selectedColor : hover == 'TZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1149.6,578.6l-2,0.8l2.3,3.6l-0.4,3.7l-1.6,0.8l0,0l0.3,2.5l1.2,1.5v2l-1.4,1.4l-2.2,3.3l-2.1,2.3l-0.6,0.1
        l-0.3,2.7l1.1,0.9l-0.2,2.7l1,2.6l-1.3,2.4l4.5,4.3l0.3,3.9l2.7,6.5l0,0l0.3,0.2l2.2,1.1l3.5,1.1l3.2,1.9l5.4,1.2l1.1,1.7l0,0
        l0.4-1.2l2.8,3.4l0.3,6.7l1.8,2.4v0.1l2.1-0.3l6.7,1.8l1.4-0.8l3.9-0.1l2.1-1.9l3.3,0.1l6.2-2.5l4.6-3.7l0,0l-2-1.4l-2.2-6.3
        l-1.8-3.9l0.4-3.1l-0.3-1.9l1.7-3.9l-0.2-1.6l-3.5-2.3l-0.3-3.6l2.8-7.9l-8-6.3l-0.4-3.7l-20.2-13l0,0l-2.8,2.8l-1.9,2.9l2.2,2.2
        l-3.2,1.6l-0.7-0.8l-3.2,0.4l-2.5,1.4l-1.6-2.4l1.1-4.5l0.2-3.8l0,0l0,0L1149.6,578.6L1149.6,578.6z"/>
      <path id="UG" fill={selected == 'UG' ? selectedColor : hover == 'UG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1167.6,545.1l-3.4,3l-4-0.1l-4.5,1.5l-3.5-1.4l-2.3,1.7l0,0l-0.3,7.5l2.3,0.8l-1.8,2.3l-2.2,1.7l-2.1,3.3l-1.2,3
        l-0.3,5.1l-1.3,2.4l-0.1,4.8l1.4,0.6l3.3-1.9l2-0.8l6.2,0.1l0,0l-0.3-2.5l2.6-3.7l3.5-0.9l2.4-1.5l2.9,1.2l0.3,0.5v-0.3l1.6-2.6
        l2.7-4.2l2.1-4.7l-2.6-7.3l-0.7-3.2L1167.6,545.1L1167.6,545.1z"/>
      <path id="UA" fill={selected == 'UA' ? selectedColor : hover == 'UA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1138.5,241l-4.8,0.5l-1.5-0.3l-1,1.4l-1.8-0.2l0,0l-4.1,0.3l-1.2,1.4l0.2,3.1l-2-0.6l-4.3,0.3l-1.5-1.5l-1.6,1.1
        l-2-0.9l-3.8-0.1l-5.6-1.5l-5-0.5l-3.7,0.2l-2.4,1.6l-2.2,0.3l3.1,5.3l-0.3,1.8l-2.3,0.7l-3.8,5.1l1.6,2.8l-1.1-0.4l-1.1,1.7
        l-0.7,2.5l2.9,1.7l0.6,1.6l1.9-1.3l3.2,0.7h3.2l2.4,1.5l1.6-0.9l3.6-0.6l1-1.5h2.1l1.1-0.9l3.2-0.6l3.9,1.9l2,0.3l2.5,1.6v2.1
        l1.9,1.1l1.1,2.6l2,1.5l-0.2,1l1,0.6l-1.2,0.5l-3-0.2l-0.6-0.9l-1,0.5l0.5,1.1l-1.1,2l-0.5,2.1l-1.2,0.7l2.4,1.1l2.2-1l2.4,1.1
        l3.3-4.6l1.3-3.4l4.5-0.8l0.7,2.4l8,1.5l1.7,1.4l-4.5,2.1l-0.7,1.2l5.8,1.8l-0.6,2.9l3,1.3l6.3-3.6l5.3-1.1l0.6-2.2l-5.1,0.4
        l-2.7-1.5l-1-3.9l3.9-2.3l4.6-0.3l3-2l3.9-0.5l-0.4-2.8l2.2-1.7l4.7-0.5l0.3-2.1l-1.8-3.4l1.3-3.2l-0.4-1.9l-7.6-2l-2.9,0.1
        l-3.6-2.9l-3.5,1l-6.6-2.2l-0.2-1.2l-2.2-2.7l-4-0.2l-0.7-1.9l0.9-1.3L1138.5,241L1138.5,241z"/>
      <path id="UY" fill={selected == 'UY' ? selectedColor : hover == 'UY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M692.5,787l-2.1-3.7l1.9-3l-3.8-4.3l-4.8-3.5l-6.2-4.1l-1.9,0.2l-6.2-4.9l-3.4,0.7l-0.5,5.1l-0.3,6.5l1.1,6.3
        l-0.9,1.4l0.4,4.2l3.9,3.5l3.6-0.2l5.4,2.7l2.7-0.6l4.2,1.2l5.3-3.5L692.5,787L692.5,787z"/>
      <path id="US" fill={selected == 'US' ? selectedColor : hover == 'US' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M116.7,450.7l2-0.9l2.5-1.4l0.2-0.4l-0.9-2.2l-0.7-0.8l-0.8-0.6l-1.9-1.1l-0.4-0.1l-0.4,0.6v1.3l-1.2,1l-0.4,0.7
        l0.4,2.3l-0.6,1.8l1.2,0.9L116.7,450.7L116.7,450.7z M116.1,440.8l0.6-0.7l-1.2-1l-1.8-0.6L113,439v0.4l0.5,0.5l0.6,1.4L116.1,440.8
        L116.1,440.8z M113.1,437.4l-2.6-0.2l-0.6,0.7l2.9,0.2L113.1,437.4z M108.4,436.5l-1.1-2.1L107,434l-1.7,0.9l0.1,0.2l0.4,1.5
        l1.8,0.2l0.4,0.1L108.4,436.5L108.4,436.5z M100.1,432.3l0.3-1.5l-1.3-0.1l-1,0.6l-0.4,0.5l1.6,1.1L100.1,432.3z M512.2,259.1h-1.6
        l-1.3,2.4h-10.1h-16.8h-16.7h-14.8h-14.7h-14.5h-15h-4.8h-14.6h-13.9l-1.6,5.1l-2.4,5.1l-2.3,1.6l1.1-5.9l-5.8-2.1l-1.4,1.2
        l-0.4,2.9l-1.8,5.4l-4.2,8.3l-4,5.6l-4,5.6l-5.4,5.8l-1.1,4.7l-2.8,5.3l-3.9,5.2l1,3.4l-1.9,5.2l1.5,5.4l1.3,2.2l-0.8,1.5l0.4,9
        l2.5,6.5l-0.8,3.5l1,1l4.6,0.7l1.3,1.7l2.8,0.3l-0.1,1.9l2.2,0.7l2.1,3.7l-0.3,3.2l6.3-0.5l7-0.7l-1,1.3l7.1,3.1l10.7,4.4H391h4.3
        l0.8-2.6h9.3l1.3,2.2l2.1,2l2.4,2.8l0.8,3.3l0.4,3.5l2.2,1.9l4,1.9l4.8-5l4.4-0.1l3.1,2.5l1.6,4.4l1,3.7l2.4,3.6l0.2,4.5l0.8,3
        l3.9,2l3.6,1.4l2.1-0.2l-0.6-2.2l0.4-3.1l1-4.4l1.9-2.8l3.7-3.1l6-2.7l6.1-4.7l4.9-1.5l3.5-0.4l3.5,1.4l4.9-0.8l3.3,3.4l3.8,0.2
        l2.4-1.2l1.7,0.9l1.3-0.8l-0.9-1.3l0.7-2.5l-0.5-1.7l2.4-1l4.2-0.4l4.7,0.7l6.2-0.8l3,1.5l2,3l0.9,0.3l6.1-2.9l1.9,1l3,5.3l0.8,3.5
        l-2,4.2l0.4,2.5l1.6,4.9l2,5.5l1.8,1.4l0.4,2.8l2.6,0.8l1.7-0.8l2-3.9l0.7-2.5l0.9-4.3l-1.2-7.4l0.5-2.7l-1.5-4.5l-0.7-5.4l0.1-4.4
        l1.8-4.5l3.5-3.8l3.7-3l6.9-4.1l1.3-2.2l3.3-2.3l2.8-0.4l4.4-3.8l6-1.9l4.6-4.8l0.9-6.5l0.1-2.2l-1.4-0.4l1.5-6.2l-3-2.1l3.2,1v-4.1
        l1.9-2.7l-1,5.3l2,2.5l-2.9,4.4l0.4,0.2l4.4-5.1l2.4-2.5l0.6-2.5l-0.9-1.1l-0.1-3.5l1.2,1.6l1.1,0.4l-0.1,1.6l5.2-4.9l2.5-4.5
        l-1.4-0.3l2.1-1.8l-0.4,0.8h3.3l7.8-1.9l-1.1-1.2l-7.9,1.2l4.8-1.8l3.1-0.3l2.4-0.3l4.1-1.1l2.4,0.1l3.8-1l1-1.7l-1.1-1.4l-0.2,2.2
        L615,306l-0.6-3.3l1.1-3.3l1.4-1.3l3.9-3.7l5.9-1.8l6-2.1l6.3-3l-0.2-2l-2.1-3.5l2.8-8.5l-1.5-1.8l-3.7,1.1l-1.1-1.7l-5.5,4.7
        l-3.2,4.9l-2.7,2.8l-2.5,0.9l-1.7,0.3l-1,1.6h-9.3h-7.7l-2.7,1.2l-6.7,4.2l0.2,0.9l-0.6,2.4l-4.6,2l-3.9-0.5l-4-0.2l-2.6,0.7
        l-0.3,1.8l0,0l-0.1,0.6l-5.8,3.7l-4.5,1.8l-2.9,0.8l-3.7,1.7l-4,0.9l-2.5-0.3l-2.7-1.3l2.7-2.4l0,0l2-2.2l3.7-3.4l0,0l0,0l0.7-2.5
        l0.5-3.5l-1.6-0.7l-4.3,2.8l-0.9-0.1l0.3-1.5l3.8-2.5l1.6-2.8l0.7-2.8l-2.7-2.4l-3.7-1.3l-1.7,2.4l-1.4,0.6l-2.2,3.1l0.4-2.1
        l-2.6,1.5l-2.1,2l-2.6,3.1l-1.3,2.6l0.1,3.8l-1.8,4l-3.3,3l-1.4,0.9l-1.6,0.7h-1.8l-0.3-0.4l-0.1-3.3l0.7-1.6l0.7-1.5l0.6-3l2.5-3.5
        l2.9-4.3l4.6-4.7h-0.7l-5.4,4l-0.4-0.7l2.9-2.3l4.7-4l3.7-0.5l4.4-1.3l3.7,0.7h0.1l4.7-0.5l-1.5-2.5l0,0l-1.2-0.2l0,0l0,0l-1.4-0.3
        l-0.4-1.7l-5.1,0.5l-5,1.4l-2.5-2.3l-2.5-0.8l3.1-3.3l-5.3,2l-4.9,2.1l-4.6,1.5l-2.1-2.1l-5.5,1.3l0.4-0.9l4.6-2.6l4.7-2.5l5.9-2.1
        l0,0l0,0l-5.3-1.6l-4.4,0.8l-3.8-1.9l-4.6-1l-3.2-0.4l-1-1L512.2,259.1L512.2,259.1z M271.6,212.2l6.9-2.8v-1.8l-2.6-0.4l-3.4,0.9
        l-6.4,2.1l-2.2,2.7l0.7,1.6L271.6,212.2z M232.9,195.8l2.3-2.3l-2.9-0.5l-5.7,1l0.8,1.6l1.6,1.1L232.9,195.8L232.9,195.8z
        M234.1,173.5l-3.1,2.2l0.4,0.5l4.2-0.4l0.3,1.1l1.7,1.2l4.9-1.2l1.2-0.6l-3.3-0.8l-1.6-1.5l-3.4,0.6L234.1,173.5L234.1,173.5z
        M359,133.3l-4.4-1.1l-10.2,2.8l-3.2-0.3l-11,2.3l-4.8,0.6l-7.8,2.5l-4.8,2.6l-8.6,2.5l-7.6,0.1l-6.3,2.9l3.2,1.7l0.7,2.3l-0.8,2.7
        l2.3,2.1l-1.2,3.5l-9.2,0.2l4.3-2.8h-3.4l-13.1,2.7l-9.1,2.3l1,3.3l-1.2,2.2l4.5,1.4l6.9-0.7l1.8,1.3l2.9-1.3l6.1-1.2h2.7l-5.9,2.1
        l1.1,1l-2.5,2.6l-5.5,1.8l-2.5-0.5l-7,2.7l-1.8-0.9l-4.1,0.4l-5.3,3l-7.6,3.1l-5.8,3.4l0.3,2.4l-4,3.3l1.4,1.4l0.5,2.7l7.2-1.1
        l0.4,2.1l-3.3,2.1l-3.6,3.5h2.8l7.2-2.3l-1.6,2.9l3.6-2.1l-0.4,3l4.8-2.2l0.4,1.1l7.2-1.8l-6.2,3.4l-5.7,4.5l-5.7,2.1l-2.3,1.2
        l-10.3,3.6l-4.9,2.4l-6.5,0.7l-8.5,3.3l-6.6,1.8l-8.1,2.8l-0.4,1l10-1.7l6-2l6.9-2l6.1-1.7l2.8,0.5l8.1-2.6l4.5-2.8l10.5-3.1
        l3.9-2.6l6.6-1.8l7.6-2.5l8.9-4.2l-0.2-2.9l11.1-4.1l7.4-3.9l9.2-3.2l-0.4,1.4l-6.7,1.8l-8.3,5.7l-3.2,3.5l6.4-1.3l6.1-1.9l6.5-1.3
        l2.9-0.3l3.5-4.1l6.3-1.2l2.6,2.5l6,2.7l6.7-0.5l5.7,2l3.2,1.1l3.3,6.1l3.7,1.7l7.1,0.2l4.1,0.4l-2.7,5.5l1.6,4.9l-3.3,5.2l2.5,1.9
        l0.6,2.2l0,0l5.1-2.9l3.1-3.7l-4.6-3.8l1.5-6.8l1.1-4.2l-1.7-2.7l-0.7-2.4l0.5-3l-6.4,1.9l-7.6,3.3l-0.2-3.9l-0.6-2.6l-2.7-1.6
        l-4.2-0.1l35.4-32.4l24.3-20.2l0,0l0,0l-3.5-0.7l-4.1-1.6l-6.5,0.8l-2.2-0.7l-7.1-0.5l-6.2-1.6l-4.8,0.5l-4.9-0.9l2-1.2l-6.3-0.3
        l-3.3,1L359,133.3L359,133.3z"/>
      <path id="UZ" fill={selected == 'UZ' ? selectedColor : hover == 'UZ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1339.8,303.1l-2.5,1.2l-5.4,4.3l-0.9,4.5h-1.9l-2.3-3l-6.6-0.2l-2.6-5l-2.5-0.1l-1.5-6.2l-7.5-4.5l-8.6,0.5
        l-5.7,0.9l-6.5-5.5l-4.8-2.3l-9.1-4.5l-1.1-0.5l-11.9,3.6l6.2,22.8l5.8-0.1l-1.6-3.1l3.8-2.2l3.3-3.6l7.8,3.3l1.9,4.9l2.3,1.3
        l5.5-0.3l2,1.2l4.3,6.4l7,4.4l4.2,3l6.2,3.2l7.7,2.7l0.8,4h2.9l4.3,1.4l1.3-6.6l-2.4-4.7l-4.2-1.6l0.6-2.8l4.4,0.3l1.5-3.5l0.5-4
        l6.4-1.5l-0.2,2.9l1.3,1.8l2.1-0.2l4.1,0.6l5.2-4.5l-7.1-3.3l-3.2,1.6l-4.6-2.3l3.1-4.1L1339.8,303.1L1339.8,303.1z"/>
      <path id="VE" fill={selected == 'VE' ? selectedColor : hover == 'VE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M642,518.9l-2.2-1.5l-2.9,0.2l-0.7-5.1l-4.1-3.2l-4.4-0.4l-1.8-3l4.8-1.9l-6.7,0.1l-6.9,0.4l-0.2,1.6l-3.2,1.9
        l-4.2-0.7l-3.1-2.9l-6,0.7l-5-0.1l-0.1-2.1l-3.5-3.5l-3.9-0.1l-1.7-4.5l-2.1,2l0.6,3l-7.1,2.6v4.8l1.6,2.2l-1.5,4.6l-2.4,0.4l-1.9-5
        l2.7-3.7l0.3-3.3l-1.7-2.9l3.3-0.8l0.3-1.5l-3.7,1.1l-1.6,3.2l-2.2,1.8l-1.8,2.4l-0.9,4.5l-1.8,3.7l2.9,0.5l0.6,2.9l1.1,1.4l0.4,2.5
        l-0.8,2.4l0.2,1.3l1.3,0.6l1.3,2.2l7.2-0.6l3.2,0.8l3.8,5.5l2.3-0.7l4,0.3l3.2-0.7l2,1.1l-1.2,3.4l-1.3,2.1l-0.5,4.6l1,4.2l1.5,1.9
        l0.2,1.5l-2.9,3.1l2,1.4l1.4,2.2l1.7,6.4l3,3.4l4.4-0.5l1.1-1.9l4.2-1.5l2.3-1l0.7-2.7l4.1-1.8l-0.3-1.4l-4.8-0.5l-0.7-4l0.3-4.3
        l-2.4-1.6l1-0.6l4.2,0.8l4.4,1.6l1.7-1.5l4-1l6.4-2.4l2.1-2.4l-0.7-1.8l-3.7-4.8l1.6-1.8v-2.9l3.4-1.1l1.5-1.2l-1.9-2.3l0.6-2.3
        L642,518.9L642,518.9z"/>
      <path id="VN" fill={selected == 'VN' ? selectedColor : hover == 'VN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1571.6,435l-5.9-1.6l-3-2.6l0.2-3.7l-5.2-1.1l-3-2.4l-4.1,3.4l-5.3,0.7h-4.3l-2.7,1.5l4,5.1l3.4,5.7l6.8,0.1l3,5.5
        l-3.3,1.7l-1.3,2.3l7.3,3.8l5.7,7.5l4.3,5.6l4.8,4.4l2,4.5l-0.2,6.4l1.8,4.2l0.1,7.7l-8.9,4.9l2.8,3.8l-5.8,0.5l-4.7,2.5l4.5,3.7
        l-1.3,4.3l2.3,4l6.6-5.9l4.1-5.3l6.1-4.1l4.3-4.2l-0.4-11.2l-4-11.7l-4.1-5.1l-5.6-4l-6.4-8.3l-5.3-6.7l0.5-4.4l3.7-6L1571.6,435z"
        />
      <path id="VU" fill={selected == 'VU' ? selectedColor : hover == 'VU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1908.6,676.9l-2.7-3.6l-0.6,1.7l1.3,2.8L1908.6,676.9L1908.6,676.9z M1906.6,667.2l-2.3-2l-0.9,4.9l0.5,1.8
        l1.2-0.4l1.3,0.8L1906.6,667.2L1906.6,667.2z"/>
      <path id="YE" fill={selected == 'YE' ? selectedColor : hover == 'YE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1271.5,466.2l-2.1-4.4l-5.2-10.5l-15.7,2.4l-5,2.9l-3.5,6.7l-2.5,1l-1.6-2.1l-2.1,0.3l-5.4-0.6l-1-0.7l-6.4,0.2
        l-1.5,0.6l-2.4-1.7l-1.2,3.1l0.7,2.7l-2.3,2.1l0.4,2.7l-0.6,1.3l0.7,2.9l-1.1,0.3l1.7,2.6l1.3,4.7l1,1.9v3.4l1.6,3.8l3.9,0.3
        l1.8-0.9l2.7,0.2l0.8-1.7l1.5-0.4l1.1-1.7l1.4-0.4l4.7-0.3l3.5-1.2l3.1-2.7l1.7,0.4l2.4-0.3l4.7-4.5l8.8-3l5.3-2.7v-2.1l0.9-2.9
        L1271.5,466.2L1271.5,466.2z"/>
      <path id="ZA" fill={selected == 'ZA' ? selectedColor : hover == 'ZA' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1148.2,713.7l-2.9-0.6l-1.9,0.8l-2.6-1.1l-2.2-0.1l-8,4.7l-5.2,4.7l-2,4.3l-1.7,2.4l-3,0.5l-1.2,3l-0.6,2l-3.6,1.5
        l-4.4-0.3l-2.5-1.8l-2.3-0.8l-2.7,1.5l-1.5,3.1l-2.7,1.9l-2.8,2.8l-4,0.7l-1.1-2.3l0.7-3.8l-3-6.1l-1.4-1l-1.1,23.6l-5,3.2l-2.9,0.5
        l-3.3-1.2l-2.4-0.5l-0.8-2.7l-2.1-1.8l-2.7,3.2l3.5,8.2v0.1l2.5,5.3l3.2,6l-0.2,4.8l-1.7,1.2l1.4,4.2l-0.2,3.8l0.6,1.7l0.3-0.9
        l2.1,2.9l1.8,0.1l2.1,2.3l2.4-0.2l3.5-2.4l4.6-1l5.6-2.5l2.2,0.3l3.3-0.8l5.7,1.2l2.7-1.2l3.2,1l0.8-1.8l2.7-0.3l5.8-2.5l4.3-2.9
        l4.1-3.8l6.7-6.5l3.4-4.6l1.8-3.2l2.5-3.3l1.2-0.9l3.9-3.2l1.6-2.9l1.1-5.2l1.7-4.7h-4.1l-1.3,2.8l-3.3,0.7l-3-3.5l0.1-2.2l1.6-2.4
        l0.7-1.8l1.6-0.5l2.7,1.2l-0.4-2.3l1.4-7.1l-1.1-4.5L1148.2,713.7L1148.2,713.7z M1128.1,766.5l-2,0.6l-3.7-4.9l3.2-4l3.1-2.5
        l2.6-1.3l2.3,2l1.7,1.9l-1.9,3.1l-1.1,2.1l-3.1,1L1128.1,766.5L1128.1,766.5z"/>
      <path id="ZM" fill={selected == 'ZM' ? selectedColor : hover == 'ZM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1149.2,626.7l-1.9-0.5l0.4-1.3l-1-0.3l-7.5,1.1l-1.6,0.7l-1.6,4.1l1.2,2.8l-1.2,7.5l-0.8,6.4l1.4,1.1l3.9,2.5
        l1.5-1.2l0.3,6.9h-4.3l-2.1-3.5l-2-2.8l-4.3-0.8l-1.2-3.4l-3.4,2l-4.5-0.9l-1.8-2.8l-3.5-0.6l-2.6,0.1l-0.3-2l-1.9-0.1l0.5,2l-0.7,3
        l0.9,3l-0.9,2.4l0.5,2.2l-11.6-0.1l-0.8,20.3l3.6,5.2l3.5,4l4.6-1.5l3.6,0.4l2.1,1.4v0.5l1,0.5l6.2,0.7l1.7,0.7l1.9-0.1l3.2-4.1
        l5.1-5.3l2-0.5l0.7-2.2l3.3-2.5l4.2-0.9l-0.3-4.5l17.1-5.2l-2.9-1.7l1.9-5.9l1.8-2.2l-0.9-5.3l1.2-5.1l1-1.8l-1.2-5.4l-2.6-2.8
        l-3.2-1.9l-3.5-1.1l-2.2-1.1l-0.3-0.2l0,0l0.5,1.1l-1,0.4L1149.2,626.7L1149.2,626.7z"/>
      <path id="ZW" fill={selected == 'ZW' ? selectedColor : hover == 'ZW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1148.2,713.7l6.2-7.2l1.6-4.6l0.9-0.6l0.8-3.7l-0.8-1.9l0.5-4.7l1.3-4.4l0.3-8.1l-2.8-2l-2.6-0.5l-1.1-1.6
        l-2.6-1.3l-4.6,0.1l-0.3-2.4l-4.2,0.9l-3.3,2.5l-0.7,2.2l-2,0.5l-5.1,5.3l-3.2,4.1l-1.9,0.1l-1.7-0.7l-6.2-0.7l1.9,5.1l1.1,1.1
        l1.6,3.7l6,7l2.3,0.7l-0.1,2.2l1.5,4.1l4.2,0.9l3.4,2.9l2.2,0.1l2.6,1.1l1.9-0.8L1148.2,713.7L1148.2,713.7z"/>
      <path id="SO" fill={selected == 'SO' ? selectedColor : hover == 'SO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1223.4,505.7l-2.6-2.7l-1.2-2.6l-1.8-1.2l-2,3.4l-1.1,2.3l2.2,3.5l2.1,3.1l2.2,2.2l18.5,7.6l4.8-0.1l-15.4,19.1
        l-7.4,0.3l-4.9,4.5l-3.6,0.1l-1.5,2l-4.8,7.2l0.2,23.2l3.3,5.3l1.3-1.5l1.3-3.4l6.1-7.7l5.3-4.8l8.3-6.4l5.6-5.1l6.4-8.7l4.7-7.1
        l4.6-9.3l3.2-8.2l2.5-7.1l1.3-6.8l1.1-2.3l-0.2-3.4l0.4-3.7l-0.2-1.7h-2.1l-2.6,2.2l-2.9,0.6l-2.5,0.9l-1.8,0.2l0,0l-3.2,0.2
        l-1.9,1.1l-2.8,0.5l-4.8,1.9l-6.1,0.8l-5.2,1.6L1223.4,505.7L1223.4,505.7z"/>
      <path id="GF" fill={selected == 'GF' ? selectedColor : hover == 'GF' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M681.4,556.2l1.8-4.7l3.5-5.8l-0.9-2.6l-5.8-5.4l-4.1-1.5l-1.9-0.7l-3.1,5.5l0.4,4.4l2.1,3.7l-1,2.7l-0.6,2.9
        l-1.4,2.8l2.4,1.3l1.8-1.8l1.2,0.3l0.8,1.8l2.7-0.5L681.4,556.2z"/>
      <path id="ES" fill={selected == 'ES' ? selectedColor : hover == 'ES' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M888.3,390.4l1-0.1v0.3l-1.2,1l-0.5,1.4l-0.4,0.6l-0.3,0.2l-0.6,0.2l-0.7-0.9l-0.4-1l-0.2-0.3l0.4-0.2h0.5l1-0.1
        l0.3-0.1L888.3,390.4z M883.3,392.7h-0.2l-0.2,0.2l-0.2,0.4l0.3,0.5l0.2,0.1h0.2l0.5-0.4v-0.2l-0.1-0.3L883.3,392.7z M880.6,389
        l-0.3-0.4h-0.7l-0.4,0.6l0.6,1.2l0.1,0.5h0.1l0.5-0.5l0.1-0.3l-0.1-0.5l0.2-0.2L880.6,389z M878.7,395.5h-0.6l0.1,0.2l0.1,0.2
        l0.7,0.4l0.6-1.1l-0.2-0.2L878.7,395.5z M901.1,389.3l-0.3,0.2l-0.1,0.6l-0.7,1.3l-0.5,1.2l-0.7,0.6l-0.7,0.2l0.1,0.1l0.7,0.1
        l0.8-0.7l1.5-0.5l0.3-1l0.3-1.1v-0.7l-0.3-0.3L901.1,389.3L901.1,389.3z M893.1,393.1L893.1,393.1L893.1,393.1h-0.2l-1.3-0.1
        l-0.2,0.6l-0.5,0.4v0.7l0.5,0.7l0.3,0.1l0.5,0.1l0.7-0.4l0.2-0.4l0.1-0.8l-0.1-0.4V393.1z M994.3,318.7l-0.3-0.1l-0.5,0.2l-0.5-0.2
        l0.1-0.3l0.1-0.2l0.1-0.1l-0.2-0.2v-0.1l0.2-0.2l-0.2-0.1l-1.3,0.4l-0.7,0.4l-2.1,1.5v0.3l0.1,0.2h0.4l0.2,0.4l0.4-0.4l0.3-0.1
        l0.3,0.1l0.3,0.2l0.1,0.6l0.1,0.2l0.6,0.1l0.9,0.4l0.4-0.2l0.5-0.3l0.2-0.6l0.3-0.5l0.3-0.5l0.3-0.4l-0.1-0.4L994.3,318.7z
        M998.6,317.1l-0.9-0.3l-1,0.1l-0.1,0.1v0.4l0.1,0.1l0.6,0.1l1.6,0.7h0.1l0.1-0.4v-0.1L998.6,317.1z M992,301.9l-6,0.8l-1.3-0.7
        l-0.2,0.1h-0.4l-0.1-0.2v-0.2l-3.7-1.8l-1.9,1.3l-9.4-2.8l-2-2.4l-8.2-0.2l-4.2,0.3l-5.4-1h-6.8l-6.2-1.1l-7.4,4.5l2,2.6l-0.4,4.4
        l1.9-1.6l2.1-0.9l1.2,3.1h3l0.9-0.8l3,0.2l1.3,3.1l-2.4,1.7l-0.2,4.9l-0.9,0.9l-0.3,3l-2.2,0.5l2,3.8l-1.6,4.3l1.8,1.9l-0.8,1.7
        l-2,2.5l0.4,2.1l4.8,1l1.4,3.7l2,2.2l2.5,0.6l2.1-2.5l3.3-2.3l5,0.1h6.7l3.8-5l3.9-1.3l1.2-4.2l3-2.9l-2-3.7l2-5.1l3.1-3.5l0.5-2.1
        l6.6-1.3l4.8-4.2L992,301.9z M903.7,386.3l-0.2,0.4l-0.6,0.2l-0.8,0.4l-0.2,0.3l-0.2,0.9l0.4,0.1l0.3-0.4l0.9-0.3l0.5-0.3l0.1-0.9
        l0.2-0.3l-0.2-0.3L903.7,386.3z M983.7,323.1l-0.2,0.3v0.3l-0.3,0.1l-0.1,0.4l0.1,0.2l0.8,0.1l0.2-0.4h0.3l0.6-0.7v-0.3l-0.3-0.2
        L983.7,323.1z M984.2,325.1l-0.1,0.2l-0.1,0.2v0.2h0.5l0.4,0.1l0.1-0.1v-0.2h-0.5L984.2,325.1z"/>
      <path id="AW" fill={selected == 'AW' ? selectedColor : hover == 'AW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M586.6,492.9l-0.1-0.1l-0.3-0.6l-0.3-0.3l-0.1,0.1l-0.1,0.3l0.3,0.3l0.3,0.4l0.3,0.1L586.6,492.9L586.6,492.9z"/>
      <path id="AI" fill={selected == 'AI' ? selectedColor : hover == 'AI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M627.9,456.2l0.1-0.2l-0.2-0.1l-0.8,0.5v0.1L627.9,456.2z"/>
      <path id="AD" fill={selected == 'AD' ? selectedColor : hover == 'AD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M985.4,301.7l0.2-0.4l-0.2-0.2l-0.7-0.2l-0.3-0.1l-0.4,0.3l-0.1,0.3l0.1,0.1v0.4l0.1,0.2h0.4L985.4,301.7
        L985.4,301.7z"/>
      <path id="AG" fill={selected == 'AG' ? selectedColor : hover == 'AG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M634.3,463.8l0.2-0.1v-0.1v-0.2l-0.1-0.1l-0.1-0.2l-0.4-0.2l-0.5,0.5v0.2l0.1,0.3l0.6,0.1L634.3,463.8L634.3,463.8z
        M634.5,460.3v-0.5l-0.1-0.2h-0.3l-0.1-0.1h-0.1l-0.1,0.1l0.1,0.6l0.5,0.3L634.5,460.3L634.5,460.3z"/>
      <path id="BS" fill={selected == 'BS' ? selectedColor : hover == 'BS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M574.4,437.3l0.2-0.6l-0.3-0.1l-0.5,0.7l-0.6,0.3h-0.3l-0.7-0.3h-0.5l-0.4,0.5l-0.6,0.1l0.1,0.1v0.2l-0.2,0.3v0.2
        l0.1,0.3l1.5-0.1l1.3-0.2l0.7-0.9L574.4,437.3z M575.2,435.3l-0.4-0.3l-0.4,0.3l0.1,0.3L575.2,435.3L575.2,435.3z M575.2,429.5
        l-0.4-0.2l-0.3,0.5l0.3,0.1l0.7-0.1l0.5,0.1l0.5,0.4l0.3-0.2l-0.1-0.1l-0.4-0.3l-0.6-0.1h-0.2L575.2,429.5L575.2,429.5z
        M568.6,430.8l0.7-0.6l0.7-0.3l0.9-1.1l-0.1-0.9l0.2-0.4l-0.6,0.1l-0.1,0.3l-0.1,0.3l0.3,0.4v0.2l-0.2,0.4l-0.3,0.1l-0.1,0.2
        l-0.3,0.1l-0.4,0.5l-0.8,0.6l-0.2,0.3L568.6,430.8L568.6,430.8z M569.8,427.6l-0.6-0.2L569,427l-0.4-0.1l-0.1,0.2v0.2l0.1,0.4
        l0.2-0.1l0.8,0.4l0.4-0.3L569.8,427.6z M565.7,426.5v-0.7l-0.4-0.5l-0.6-0.4l-0.1-1.2l-0.3-0.7l-0.2-0.6l-0.4-0.8v0.5l0.1,0.1
        l0.1,0.6l0.4,0.9l0.1,0.4l-0.1,0.4l-0.4,0.1l-0.1,0.2l0.5,0.3l0.8,0.3l0.5,1.3L565.7,426.5L565.7,426.5z M561.6,423l-0.5-0.3
        l-0.2-0.3l-0.7-0.7l-0.3-0.1l-0.2,0.4l0.4,0.1l0.9,0.7l0.4,0.2L561.6,423L561.6,423z M568.9,419l-0.1-0.3h-0.1l-0.3,0.1l-0.3,0.9
        h0.3L568.9,419L568.9,419z M551.3,417.9l-0.2-0.3l-0.3,0.2h-0.5l-0.2,0.1h-0.4l-0.3,0.2l0.4,0.8l0.3,0.3l0.1,1l0.2,0.1l-0.1,0.7
        l1.1,0.1l0.4-0.8V420v-0.1v-0.2v-0.2v-0.9l-0.3-0.5l-0.4,0.6l-0.4-0.3l0.6-0.4L551.3,417.9L551.3,417.9z M564.2,418.2l-1-1.4v-0.2
        l-0.5-1.5l-0.3-0.1l-0.1,0.1l-0.1,0.2l0.4,0.4v0.4l0.3,0.2l0.4,1.1l0.4,0.4l-0.1,0.3l-0.4,0.3l-0.1,0.2h0.1l0.6-0.1h0.4L564.2,418.2
        L564.2,418.2z M553.7,413l0.5-0.2l0,0l-0.3-0.2h-0.7l-0.4,0.1l-0.2,0.2l0.1,0.1l0.4,0.1L553.7,413L553.7,413z M551.3,415l-0.5-0.6
        l-0.3-0.9l-0.2-0.4l0.1-0.5l-0.3-0.4l-0.6-0.4l-0.3,0.1l0.1,1.1l-0.2,0.6l-0.8,1.1l0.1,0.4l0,0l0.1,0.2l-0.5,0.4v-0.3l-0.6,0.1
        l0.3,0.5l0.6,0.4l0.3,0.1l0.3-0.2v0.5l0.3,0.4l0.1,0.4l0.3-0.3l0.6-0.2l0.2-0.2l0.7-0.4v-0.2l0.1-0.6L551.3,415L551.3,415z M558,410
        l-0.3-0.5l-0.1,0.1l-0.1,0.4l-0.3,0.4l0.5-0.1l0.4,0.1l0.6,0.5l0.7,0.2l0.3,0.6l0.6,0.6v0.6l-0.4,0.6l-0.1,0.7l-0.6,0.1l0.1,0.1
        l0.3,0.3l0.1,0.4l0.2,0.2v-0.7l0.3-0.8l0.4-1.3l-0.1-0.3l-0.3-0.3l-0.7-0.9l-0.7-0.3L558,410L558,410z M549.2,402.1l-0.5-0.4
        l-0.2,0.4v0.1l-0.1,0.3l-0.5,0.4l-0.5,0.1l-0.7-0.6l-0.2-0.1l0.8,1.1l0.3,0.1h0.4l0.9-0.3l1.6-0.5l1.7-0.2l0.1-0.2l-0.1-0.3
        l-0.8,0.2l-1-0.1l-0.2,0.2h-0.4L549.2,402.1z M555.3,407.3l0.2-0.3l0.4-1.8l0.8-0.6l0.1-1.2l-0.5-0.5l-0.4-0.2l-0.1-0.2l0.1-0.2
        l-0.2-0.1l-0.3-0.2l-0.4-0.6l-0.4-0.4l-0.7-0.1l-0.6-0.1l-0.4-0.1l-0.5,0.3h0.8l1.5,0.3l0.7,1.5l0.5,0.4l0.1,0.4l-0.2,0.4v0.4
        l-0.3,0.5l-0.1,0.8l-0.3,0.4l-0.7,0.5l0.4,0.2l0.3,0.6L555.3,407.3L555.3,407.3z"/>
      <path id="FR" fill={selected == 'FR' ? selectedColor : hover == 'FR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1012.2,290.9l2.7,0.8l-0.5,2.7l-0.1,0.1l-0.3-0.2l-0.5,0.6l0,0.3l-3.6,2.6l-10-1.6l-7.4,2l-0.5,3.7l-6,0.8
        l-1.3-0.7l0.7-0.3l0.2-0.4l-0.2-0.2l-0.7-0.2l-0.3-0.1l-0.4,0.3l-0.1,0.3l0.1,0.1v0.2l-3.7-1.8l-1.9,1.3l-9.4-2.8l-2-2.4l2.7-3.7
        l1-12.3l-5.1-6.5l-3.6-3.1l-7.5-2.4l-0.4-4.6l6.4-1.3l8.2,1.6l-1.4-7l4.6,2.6l11.3-4.8l1.4-5.1l4.3-1.2l0.7,2.2l2.2,0.1l2.4,2.4
        l3.4,2.9l2.5-0.4l4.4,2.8l0,0l1.1,0.5l1.4-0.1l2.4,1.6l7.1,1.2l-2.3,4.2l-0.5,4.5l-1.3,1l-2.3-0.6l0.2,1.6l-3.5,3.5v2.8l2.4-0.9
        l1.8,2.7l0,0l-0.2,1.7l1.6,2.4l-1.7,1.8L1012.2,290.9z M1025.6,304.3l-1-6l-0.6,1.6l-2.7,1.1l-0.7,4.3l3,3.7L1025.6,304.3z"/>
      <path id="MC" fill={selected == 'MC' ? selectedColor : hover == 'MC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1013.5,295.2l0-0.3l0.5-0.6l0.3,0.2L1013.5,295.2z"/>
      <path id="BM" fill={selected == 'BM' ? selectedColor : hover == 'BM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M630.2,366.8l0.4-0.6h-0.1l-0.5,0.5l-0.6,0.2l0.1,0.1h0.1L630.2,366.8z"/>
      <path id="BB" fill={selected == 'BB' ? selectedColor : hover == 'BB' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M644.9,488.9l0.4-0.4l-0.3-0.3l-0.6-0.8l-0.3,0.1v1l0.1,0.3l0.5,0.3L644.9,488.9L644.9,488.9z"/>
      <path id="KM" fill={selected == 'KM' ? selectedColor : hover == 'KM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1221.1,650.5l-0.4-0.4h-0.4v0.2l0.1,0.4l1.1,0.2L1221.1,650.5L1221.1,650.5z M1225,649L1225,649l-0.3,0.1l-0.1,0.2
        l-0.1,0.3h-0.3h-0.2h-0.4l0.8,0.5l0.5,0.5l0.2,0.2l0.1-0.2l0.1-0.7L1225,649L1225,649z M1219.4,647.9l0.2-0.3l-0.2-0.7l-0.4-0.8
        l0.1-1.4l-0.2-0.2h-0.3l-0.1,0.1l-0.1,0.3l-0.3,2l0.4,0.6l0.3,0.1L1219.4,647.9L1219.4,647.9L1219.4,647.9z"/>
      <path id="CV" fill={selected == 'CV' ? selectedColor : hover == 'CV' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M841.4,477.6l0.1-0.4l-0.2-0.6l-0.3-0.1l-0.6,0.4l-0.1,0.3l0.1,0.3l0.3,0.3l0.3,0.1L841.4,477.6L841.4,477.6z
        M847.7,475.9l0.4-0.2V475l-0.1-0.3h-0.4l-0.2,0.4v0.1v0.4L847.7,475.9L847.7,475.9L847.7,475.9z M846.3,476.7l-0.5-0.9l-0.3-0.1
        l-0.6-0.7v-0.3l-0.3-0.1v0.2v0.4l-0.2,0.5v0.5l0.4,0.8l0.4,0.2l0.7,0.1L846.3,476.7L846.3,476.7z M849.4,468.9v0.5l-0.3,0.7l0.5,0.3
        l0.3,0.1l0.6-0.4l0.2-0.5l-0.1-0.3l-0.3-0.3l-0.3-0.1l-0.1,0.1L849.4,468.9L849.4,468.9z M843,466.4l-1-0.1l-0.6-0.2h-0.1v0.3
        l0.4,0.8l0.2-0.5l0.2-0.1l0.8,0.2l0.4-0.1l-0.1-0.1L843,466.4L843,466.4z M849.7,466.2l-0.1-0.5V465h-0.2l-0.3,0.2l0.1,0.7l0.1,0.1
        l0.2,0.5L849.7,466.2L849.7,466.2z M838.6,465.2V465l-0.3-0.5l-0.3,0.1l-0.4,0.2l-0.1,0.3l0.4,0.2h0.2L838.6,465.2L838.6,465.2z
        M837.1,464.3l0.8-0.6l0.2-0.3l-0.2-0.5l-0.5-0.1l-1.2,0.6l-0.1,0.2l0.1,0.3l0.1,0.5l0.2,0.1L837.1,464.3L837.1,464.3z"/>
      <path id="KY" fill={selected == 'KY' ? selectedColor : hover == 'KY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M527,449.1l-0.1-0.3l-0.1,0.1v0.6h0.5h0.2l0.3-0.2h0.6l-0.1-0.2l-0.8-0.1l-0.1,0.1l-0.2,0.1L527,449.1L527,449.1z
        M535,446.8L535,446.8l-0.1-0.1h-0.1l-0.3,0.1h-0.1h-0.1l-0.1,0.1l-0.1,0.1h0.2l0.4-0.2H535L535,446.8L535,446.8z M535.8,446.7
        l0.5-0.2l0,0l-0.1-0.1h-0.1l-0.1,0.1h-0.1l-0.5,0.3h0.2L535.8,446.7L535.8,446.7z"/>
      <path id="DM" fill={selected == 'DM' ? selectedColor : hover == 'DM' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M635.8,475.1l0.3-0.7l-0.1-1l-0.2-0.4l-0.8-0.3v0.2l-0.1,0.5l0.3,0.8l0.1,1.1L635.8,475.1z"/>
      <path id="FK" fill={selected == 'FK' ? selectedColor : hover == 'FK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M690.3,902.7l-0.1-0.3l-0.4-0.2l-0.2-0.1l0.1,0.2l0.1,0.3l0.1,0.2l0.2,0.1L690.3,902.7L690.3,902.7z M695.8,901.4
        L695.8,901.4l-0.3-0.1l-0.1,0.2l0.2,0.3l0.4,0.1L695.8,901.4L695.8,901.4z M682.9,900l-0.1,0.2l-0.4,0.1l0.2,0.3l0.6,0.4h0.4
        l0.1-0.3l-0.1-0.6h-0.3L682.9,900L682.9,900z M685.7,898l-0.9-0.3l-0.4-0.3h-0.3l0.4,0.4l0.1,0.2l0.1,0.2l0.6,0.3l0.6,0.3l0.4,0.3
        l-0.1,0.1l-0.8,0.3h-0.3l-0.2,0.1l0.4,0.2l0.6-0.1l0.2-0.1h0.2l0.3,0.1v0.2l-0.1,0.2l-0.2,0.2l-0.4,0.3l-0.6,0.4h-0.8l-0.7,0.7
        l0.9,0.5l0.7,0.3h0.9v-0.1l0.2-0.1h0.3l0.1-0.1l0.2-0.4v-0.6h0.2l0.3,0.1l0.7-0.1l0.3-0.1l0.6-0.9l0.4-0.8l0.2-0.4l0.3-0.2l0.1-0.2
        l0.1-0.3l0.3-0.2v-0.3l-0.4-0.2l-0.3-0.2l-0.3,0.3l-0.2-0.1l-0.9,0.3h-0.4l-0.3-0.2l-0.4-0.1l-0.4,0.1l-0.5,0.5L685.7,898L685.7,898
        z M686.4,897.6l0.1-0.3l-0.1-0.2l-0.5-0.2h-0.5l0.2,0.5l0.2,0.2H686.4z M692.3,896.9h-0.4l0.4,0.5l-0.8,0.8l0.2,0.6l0.3,0.4l0.1,0.2
        l-0.1,0.1l-0.4,0.1l-0.3,0.1l-0.2,0.3l-0.9,0.9l0.2,0.2l-0.3,0.7l0.2,0.3l0.8,0.7l0.8,0.4v-0.7l0.4-0.1l0.4,0.2l0.4-0.2l-0.9-1h0.3
        l2.5,0.5l-0.1-0.4l-0.1-0.2l-0.3-0.4l1.5-0.4l0.5-0.3l0.2-0.3l0.6-0.1l0.8-0.3l-0.1-0.1l0.1-0.3l-0.4-0.2l-0.5-0.1l0.1-0.3l0.5-0.1
        l-0.8-0.7l-0.3-0.1l-1,0.1l-0.3,0.1v0.2l0.1,0.3l0.3,0.3l0.1,0.2l-0.2-0.1l-1.1-0.4l-0.2-0.1l-0.2-0.4l0.2-0.1l0.3,0.1l0.1-0.3
        l-0.4-0.3l-0.4-0.1l-0.9,0.1L692.3,896.9L692.3,896.9z"/>
      <path id="FO" fill={selected == 'FO' ? selectedColor : hover == 'FO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M947,186.9v-0.3l-0.1-0.3v-0.2h-0.1l-0.5-0.1l-0.1-0.2h-0.1v0.2l0.1,0.4l0.5,0.4L947,186.9l0.1,0.1L947,186.9
        L947,186.9z M947.5,184.8v-0.1l-0.2-0.2l-0.5-0.2l-0.2-0.1l-0.2,0.1v0.2l0.1,0.1l0.4,0.1l0.4,0.3h0.1L947.5,184.8L947.5,184.8z
        M945.1,182.9l-0.2-0.1l-0.5,0.1h-0.3l0.1,0.3l0.6,0.2h0.3h0.3l0.2-0.1l-0.1-0.2L945.1,182.9L945.1,182.9z M947.6,182.4l-0.8-0.2
        l-0.6-0.3l-1,0.1l0.7,1.1l0.8,0.7l0.4,0.2v-0.1v-0.2l-0.4-0.5l-0.1-0.1V183l0.1-0.1h0.2l0.3,0.2h0.2L947.6,182.4L947.6,182.4z
        M948.6,182.2l-0.3-0.2l-0.4-0.4v0.5v0.3v0.1h0.1l0.3,0.1L948.6,182.2L948.6,182.2z"/>
      <path id="GD" fill={selected == 'GD' ? selectedColor : hover == 'GD' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M632.1,495.7l0.5-0.2l0.2-1.1l-0.3-0.1l-0.3,0.3l-0.3,0.5v0.4l-0.2,0.3L632.1,495.7L632.1,495.7z"/>
      <path id="HK" fill={selected == 'HK' ? selectedColor : hover == 'HK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1604.9,430.9v-0.2v-0.2l-0.4-0.2h-0.3l0.1,0.2l0.4,0.5L1604.9,430.9L1604.9,430.9z M1603.6,430.9l-0.1-0.5l0.2-0.3
        l-0.9,0.3l-0.1,0.3v0.1l0.2,0.1H1603.6L1603.6,430.9z M1605.2,429.7l-0.1-0.3l-0.2-0.1l-0.1-0.3l-0.1-0.2l0,0l-0.3-0.1l-0.2-0.1
        h-0.4l-0.1,0.1h-0.2l-0.2,0.2l0,0v0.2l-0.5,0.4v0.2l0.3,0.2l0.5-0.1l0.6,0.2l0.8,0.3v-0.2v-0.3L1605.2,429.7L1605.2,429.7z"/>
      <path id="KN" fill={selected == 'KN' ? selectedColor : hover == 'KN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M629.9,463.2v-0.3l-0.2-0.2h-0.3v0.5l0.2,0.2L629.9,463.2z M629.4,462.5l-0.1-0.2l-0.1-0.1l-0.2-0.4l-0.4-0.4
        l-0.2,0.1l-0.1,0.2v0.1l0,0l0.3,0.3l0.4,0.1l0.2,0.4L629.4,462.5L629.4,462.5z"/>
      <path id="LC" fill={selected == 'LC' ? selectedColor : hover == 'LC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M637.4,484.2l0.1-1.2l-0.1-0.5l-0.2,0.1l-0.3,0.4l-0.4,0.6l-0.1,0.3v0.6l0.6,0.4L637.4,484.2L637.4,484.2z"/>
      <path id="LI" fill={selected == 'LI' ? selectedColor : hover == 'LI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1024.4,273.6v-0.2l0.1-0.2l-0.1-0.1l-0.1-0.2l-0.1-0.1v-0.2l-0.1-0.1v-0.2l-0.1-0.1l-0.2,0.6v0.5l0.1,0.2h0.1
        L1024.4,273.6L1024.4,273.6z"/>
      <path id="MV" fill={selected == 'MV' ? selectedColor : hover == 'MV' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1389.1,551.6L1389.1,551.6l0.1-0.3l-0.1-0.1h-0.1l-0.1,0.2v0.1v0.1H1389.1z M1389.4,545.7l0.1-0.2v-0.1v-0.1v-0.1
        v-0.1l-0.1,0.1l-0.1,0.2v0.1l-0.1,0.1v0.1H1389.4L1389.4,545.7z"/>
      <path id="MT" fill={selected == 'MT' ? selectedColor : hover == 'MT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1053.6,344l-0.2-0.2l-0.5-0.5l-0.5-0.1l0.1,0.6l0.4,0.4h0.5L1053.6,344L1053.6,344z M1052.2,342.8L1052.2,342.8
        v-0.2l-0.3-0.1l-0.4,0.1l0.1,0.1l0.3,0.2L1052.2,342.8z"/>
      <path id="MS" fill={selected == 'MS' ? selectedColor : hover == 'MS' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M631.8,465.7l-0.1-0.5h-0.1l-0.2,0.4v0.3l0.3,0.1L631.8,465.7z"/>
      <path id="MU" fill={selected == 'MU' ? selectedColor : hover == 'MU' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1294.7,702.5l0.3-0.3l0.2-0.4l0.3-0.3l0.1-0.7l-0.2-0.8l-0.4-0.7l-0.5,0.1l-0.3,0.4l-0.2,0.5l-0.5,0.3l-0.1,0.3
        l-0.2,0.7l-0.1,0.4l-0.2,0.1v0.2l0.3,0.3l0.8,0.1L1294.7,702.5L1294.7,702.5z"/>
      <path id="NC" fill={selected == 'NC' ? selectedColor : hover == 'NC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1897.3,716.1v-0.3l-0.4-0.2l-0.2,0.5v0.1l0.2,0.1h0.2L1897.3,716.1L1897.3,716.1z M1901.9,708.5L1901.9,708.5
        l-0.1-0.4l0.1-0.2l-0.4,0.2l-0.6,0.2l0.1,0.8l-0.1,0.4l0.3,0.1l0.1,0.3h0.2l0.7-0.2l0.3-1.1h-0.4L1901.9,708.5L1901.9,708.5z
        M1898.9,706.8l0.3-0.5l0.1-0.2l-0.2-0.7l-0.3-0.3l0.3-1l-0.1-0.2l-0.4-0.2l-0.9,0.3l-0.1,0.2l0.5,0.1l0.2,0.2l-0.5,0.7l-0.5,0.1
        l0.1,0.5l0.2,0.4l0.7,0.2l0.3,0.4H1898.9z M1895,703.9l0.3-0.3l0.3-0.4l-0.1-0.1v-0.3l0.2-0.4l0.3-0.1l-0.2-0.2l-0.2-0.1v0.3
        l-0.3,0.7l-0.1,0.3l-0.5,0.6H1895L1895,703.9z M1882.7,701l-0.6-0.7l-0.1,0.2l-0.1,0.4v0.3l0.3,0.2l0.1,0.2l-0.1,0.5v0.4l0.6,0.9
        l0.1,0.7l0.3,0.6l0.5,0.5l0.4,0.5l0.8,1.4l0.2,0.5l0.4,0.3l1,1.2l0.4,0.4l0.4,0.2l0.9,0.7l0.6,0.3l0.3,0.5l0.6,0.3l0.8,0.4l0.1,0.2
        v0.3l0.1,0.3l0.5,0.4l0.6,0.3l0.1,0.2l0.1,0.2l0.3-0.1l0.3,0.1l0.9,0.7l0.4-0.1h0.3l0.5-0.2l0.3-0.4l-0.1-1.1l-0.5-0.5l-0.7-0.4
        l-0.4-0.5l-0.4-0.5l-0.8-1l-1.1-1l-0.5-0.2l-0.3-0.4l-0.3-0.1l-0.2-0.3l-0.5-0.3l-0.3-0.6l-0.6-0.6l-0.1-0.3l0.1-0.3l-0.1-0.3
        l-0.4-0.3l-0.2-0.5l-0.2-0.3l-0.4-0.2l-0.7-0.4l-1.6-1.9l-0.7-0.6l-0.7,0.2L1882.7,701L1882.7,701z M1860.7,695l0.2-0.4l0.1-0.8
        l-0.2,0.4l-0.2,1L1860.7,695z"/>
      <path id="NR" fill={selected == 'NR' ? selectedColor : hover == 'NR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1915,575.5v-0.2h-0.1h-0.1l-0.1,0.2l0.1,0.1l0.1,0.1L1915,575.5L1915,575.5z"/>
      <path id="PN" fill={selected == 'PN' ? selectedColor : hover == 'PN' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M274.2,727.4v-0.2l-0.1-0.2l-0.2-0.1l-0.1,0.1l0.1,0.2l0.2,0.2H274.2L274.2,727.4z"/>
      <path id="PR" fill={selected == 'PR' ? selectedColor : hover == 'PR' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M600.8,457.3v-0.1l0,0h0.1v-0.1l0.1-0.1l0,0v-0.1h-0.1l0,0h-0.3h-0.1v0.1v0.1l0.2,0.1l0,0L600.8,457.3L600.8,457.3
        L600.8,457.3z M614.4,457l0.7-0.2v-0.1l-0.4-0.1h-0.6l-0.5,0.2l0.1,0.2h0.2H614.4z M610.7,454.8l-0.1-0.2h-0.2l-3.5-0.1l-1.3-0.2
        l-0.3,0.1l-0.3,0.1l-0.1,0.4l-0.2,0.2l-0.3,0.2l0.1,0.3l0.1,0.2l0.2,0.4l-0.1,0.5l-0.2,1l0.3,0.2l0.7-0.1l0.3,0.1l0.3,0.1l0.4-0.1
        l0.4-0.2l0.9,0.1l0.5-0.1l0.6,0.3l0.4-0.1l0.2,0.1h0.3h0.6l0.9-0.2l0.8-0.5l0.3-0.5l0.4-0.3l0.6-0.4v-0.9l-0.7-0.1l-0.6-0.3
        l-1.1-0.1h-0.1l0.1,0.2h-0.1L610.7,454.8L610.7,454.8z"/>
      <path id="PF" fill={selected == 'PF' ? selectedColor : hover == 'PF' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M213.2,704.9l-0.1-0.3l-0.2-0.3l-0.1,0.1l0.1,0.1l0.2,0.3v0.2L213.2,704.9z M222.5,690.2l-0.2-0.2l-0.4-0.2
        l-0.2-0.1l-0.2-0.1l-0.1,0.1l0.1,0.1h0.1l0.3,0.2l0.3,0.1L222.5,690.2L222.5,690.2L222.5,690.2L222.5,690.2z M198,689.1l-0.6-0.3
        l0.1,0.2l0.4,0.2l0.2,0.1L198,689.1L198,689.1z M218.5,688.9l-0.4-0.5h-0.3L218.5,688.9L218.5,688.9z M196.9,687.9l-0.4-0.4
        l-0.2-0.3l-0.3-0.1l0.1,0.1l0.4,0.4l0.3,0.4l0.2,0.1L196.9,687.9z M196.6,685.8l-0.1-0.1l0,0v-0.3l0.2-0.3l0.6-0.4v-0.1l0,0
        l-0.2,0.1l-0.4,0.2l-0.2,0.2l-0.1,0.2l-0.1,0.3l0.1,0.2l0.1,0.1h0.2L196.6,685.8L196.6,685.8z M149.2,684.7l-0.2-0.6l-0.3-0.5
        l-0.8-0.1l-0.5,0.2l-0.1,0.2l0.1,0.4l0.5,0.7l0.5,0.1l0.8-0.1l0.4,0.6l0.2,0.1l0.4,0.1l0.1-0.3l-0.2-0.5L149.2,684.7L149.2,684.7z
        M146.3,683.8l0.1-0.4l-0.2-0.1h-0.5v0.2l0.1,0.2l0.1,0.1l0.3,0.2L146.3,683.8L146.3,683.8z M136.6,679.5h0.2l-0.4-0.6l-0.3-0.2v0.1
        v0.7l0.3,0.1L136.6,679.5z M180.5,677.9h-0.2H180h-0.1l0.5,0.1l0.4,0.2L180.5,677.9L180.5,677.9z M179.8,678l-0.3-0.1l-0.3-0.2h-0.3
        l0.7,0.3H179.8L179.8,678z M136,678.1l0.1-0.2l-0.1-0.1l-0.4-0.2l0.1,0.3v0.2H136L136,678.1L136,678.1z M168.8,676.1l-0.3-0.4
        l-0.2-0.3l-0.2-0.4l-0.4-0.5l0.1,0.3l0.1,0.2l0.2,0.2l0.2,0.4l0.1,0.2l0.3,0.4h0.1L168.8,676.1L168.8,676.1z M185,674.6l0.1-0.5
        h-0.2L185,674.6L185,674.6L185,674.6z M170.6,673l-0.6-0.6h-0.1l0.1,0.2l0.5,0.5l0.1,0.2V673L170.6,673z M201.4,639.1l0.1-0.2v-0.2
        l-0.1-0.1l-0.3-0.1l0.1,0.7L201.4,639.1L201.4,639.1z M198.7,635.4l-0.1-0.2h-0.2l-0.1,0.1v0.5L198.7,635.4L198.7,635.4z
        M198.8,633.8l-0.8,0.5l0.2,0.4l0.4,0.1l0.2-0.2l0.8-0.1l0.3-0.4l-0.3,0.1L198.8,633.8L198.8,633.8z M192.7,632.1l0.2-0.5l-0.2-0.1
        l-0.4,0.2v0.2l0.3,0.4L192.7,632.1L192.7,632.1z M195.3,629l0.3-0.1v-0.1l-0.2-0.2l-0.3-0.1l-0.1,0.1l-0.1,0.2l0.1,0.3L195.3,629
        L195.3,629z M192.4,628.9l0.1-0.3v-0.2l-0.1-0.2l-0.9-0.2l-0.1,0.1v0.4l0.2,0.5h0.3L192.4,628.9z"/>
      <path id="SG" fill={selected == 'SG' ? selectedColor : hover == 'SG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1561,563.7l0.1-0.2l-0.2-0.2l-0.3-0.1l-0.5-0.2l-0.6,0.1l-0.3,0.6l0.9,0.4L1561,563.7L1561,563.7z"/>
      <path id="SB" fill={selected == 'SB' ? selectedColor : hover == 'SB' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1909.1,646.4l-0.2-0.2l-0.1-0.4h-0.3l-0.3,0.1l0.2,0.6h0.2L1909.1,646.4L1909.1,646.4z M1873.5,647.2l-0.1-0.2
        l-0.5-0.4l-1.9-1.3l-0.4-0.1l-0.1,0.1l-0.1,0.3l0.1,0.2l0.5,0.1v0.1l0.3,0.2l0.7,0.2l0.4,0.3l0.1,0.5l0.3,0.1l0.3,0.1L1873.5,647.2
        L1873.5,647.2z M1905.5,640.6L1905.5,640.6l0.2-0.4l-0.2-0.1l-0.5-0.1l-0.7,0.1l-0.3,0.2l-0.2,0.3h-0.2v0.2l0.1,0.4l0.2-0.1l0.2,0.1
        l0.5-0.5h0.3h0.1L1905.5,640.6L1905.5,640.6z M1881.1,638.3l-0.1-0.2l-0.2-0.1l-0.9-0.7l-0.5-0.2h-0.5l-0.1,0.5v0.3h0.6l0.4,0.2v0.6
        l0.2,0.2v0.5l1.2,0.9l0.7,0.4l0.7,0.1l0.4,0.2l0.5-0.1l0.5,0.2l0.4-0.1l-0.4-0.3v-0.4l-0.5-1.3l-0.3-0.3l-0.5,0.1l-0.5-0.2h-0.4
        L1881.1,638.3L1881.1,638.3z M1880.7,633.4l-0.6-1.6l-0.2-0.1l0.1,0.6l0.1,0.4l-0.1,0.5l-0.1,0.6l0.2,0.2l0.2-0.2l0.4,0.5v-0.2
        V633.4z M1870.9,631.2l-0.3-0.1l-0.4,0.3l-0.1,0.3l-0.1,0.7v0.4l0.3,0.7l0.3,0.5l0.3,0.3l0.2,0.2l0.9,0.1l1.7,0.1l0.9,0.4l0.9,0.2
        l0.4-0.1l0.5-0.2l0.1-0.1l-0.1-0.6l-0.2-0.3l-0.4-0.2l-0.2-0.6l-0.5-0.4l-0.9-0.7h-1.6l-0.6,0.1L1870.9,631.2L1870.9,631.2z
        M1873.5,629.4l-0.5,0.2v0.3l0.4,0.1l0.4,0.2l0.1,0.3l0,0l0.2-0.1l0.4,0.2l0.2-0.3l-0.4-0.5l-0.4-0.3h-0.1L1873.5,629.4
        L1873.5,629.4z M1867.9,630.2l0.3-0.2v-0.4h-0.3l-0.1-0.2h-0.2l-0.3,0.2l-0.2,0.3l0.1,0.2h0.4L1867.9,630.2L1867.9,630.2
        L1867.9,630.2z M1859.5,627.9l-0.1-0.2l-0.3-0.2h-0.2l-0.5,0.1l0.1,0.1l0.6,0.3l0.3,0.1L1859.5,627.9L1859.5,627.9z M1862.6,628.3
        l0.3-0.2l-0.1-0.2l-0.1-0.5l-0.4,0.7l0.1,0.2H1862.6z M1862.1,627.4v-0.2V627l-0.2-0.1l0.4-0.3l-0.1-0.1l-0.6-0.2l-0.2,0.2l-0.2,0.1
        l-0.1,0.1l-0.1,0.1l-0.1,0.5l0.2,0.4l0.4,0.2L1862.1,627.4L1862.1,627.4z M1858.1,627.6l-0.3-0.4l0.1-0.5l0.2-0.1l0.2-0.5l-0.1-0.4
        l-0.2,0.1l-0.7,0.6l-0.1,0.3l0.6,0.8L1858.1,627.6L1858.1,627.6L1858.1,627.6z M1871.1,626.3l-0.2-0.4v-0.2l-0.3-0.2l-0.2,0.1
        l-0.1,0.3l0.1,0.2l0.4,0.3L1871.1,626.3L1871.1,626.3z M1877.1,625.1h-0.2l-0.1,0.1h-0.2h-0.3l-0.1,0.2l0.6,1.1l-0.3,0.5l0.4,2.2
        l0.4,1.2l0.8,0.8v0.2l0.8,0.5l0.6,1.3l0.2,0.1l0.1-0.2v-0.6l-0.5-1.1l0.1-0.8l-0.2-0.3V630l-0.2-0.8l-0.6-0.7l-0.3-0.1l-0.2-0.3
        l0.2-0.6l0.2-0.2l0.1-0.3L1877.1,625.1L1877.1,625.1z M1860.5,624.6l-0.6-0.2l-0.2-0.3v-1l-0.6-0.3l-0.3,0.2l-0.6,0.7l-0.2,0.4
        l-0.5,0.3l-0.1,0.3v0.4l0.4,0.1l0.3-0.4l0.9-0.1l0.3,0.1v0.4l0.1,0.7l0.3,0.3l0.5,0.2l0.4,0.6l0.1-0.3h0.2l0.2-0.4l-0.3-1.2
        L1860.5,624.6L1860.5,624.6z M1854,624.2l0.1-0.5l-0.1-0.9l-0.2,0.1v0.2l-0.1,0.4L1854,624.2L1854,624.2z M1857.2,623.8l0.2-0.2
        v-0.4v-0.5l-0.2-0.4l-0.2-0.2l-0.5,0.1l-0.4,0.5v0.5l0.4,0.6L1857.2,623.8L1857.2,623.8L1857.2,623.8z M1854.6,622.6l0.2-0.3
        l0.5-0.7l0.1-0.3l-0.5-0.2l-0.4-0.5l-0.4-0.2l-0.3,0.4v0.4l0.5,0.6l-0.1,0.4l0.2,0.1l0.1,0.4L1854.6,622.6L1854.6,622.6z
        M1872.1,626.5l-0.1-0.5l-0.3-0.4l0.4-0.5l-2.2-1.9l-0.3-0.2l-0.4-0.1l-0.5-0.4l-0.5-0.1l-0.5-0.4l-0.2-0.3l-0.6-0.4l-0.6-0.8
        l-1.5-0.3l0.1,0.2l0.4,0.4l0.1,0.7l0.5,0.4l0.5,0.6l0.2,0.1l0.2,0.2l0.4,0.5l0.8,0.4l0.8,0.6l0.3,0.1l0.3,0.3l1.5,0.7l0.5,0.7
        L1872.1,626.5L1872.1,626.5L1872.1,626.5z M1850.3,617.3l0.2-0.3l-0.7-0.5l-0.2,0.3l-0.2,0.5l0.4,0.2L1850.3,617.3L1850.3,617.3z
        M1859.4,618.8L1859.4,618.8l-0.4-0.1l-0.4-0.2l-0.7-0.8l-0.2-0.3l-0.2-1l-0.4-0.4l-1.4-0.8l-0.8-0.8l-0.7-0.2l-0.2,0.2v0.5l0.2,0.3
        l1,0.9l1.1,1.7l1,1l0.8,0.1h0.4v0.1l0.1,0.2l0.5,0.2l0.5-0.4L1859.4,618.8L1859.4,618.8z"/>
      <path id="ST" fill={selected == 'ST' ? selectedColor : hover == 'ST' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1014.1,571.4l0.5-0.8v-0.5l-0.3-0.5h-0.4l-0.5,0.4l-0.3,0.4v0.3l0.1,0.7l0.1,0.3l0.3,0.2L1014.1,571.4
        L1014.1,571.4z M1018.4,562.2l0.2-0.4v-0.2l-0.1-0.1l-0.1-0.1l-0.2,0.1l-0.3,0.5l0.1,0.2l0.2,0.2L1018.4,562.2L1018.4,562.2z"/>
      <path id="SX" fill={selected == 'SX' ? selectedColor : hover == 'SX' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M627.1,457.2L627.1,457.2l0.2,0.2l0.3,0.1l0.1-0.1v-0.2H627.1z"/>
      <path id="SC" fill={selected == 'SC' ? selectedColor : hover == 'SC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1288.5,602l-0.5-0.8l-0.4,0.3l0.2,0.3l0.3,0.2l0.1,0.4l0.3,0.2V602L1288.5,602z"/>
      <path id="TC" fill={selected == 'TC' ? selectedColor : hover == 'TC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M578.7,433.1l-0.1,0.4v0.2l0.2,0.1l0.6-0.1l0.1-0.1l0.2-0.1v-0.1l-0.4,0.1L578.7,433.1z M582.3,433.7l0.2-0.2
        l-0.2-0.2l-0.7-0.2l-0.2,0.1v0.3h0.6L582.3,433.7L582.3,433.7L582.3,433.7z M581.2,433.2l-0.1-0.1l-0.1-0.6h-0.5v0.2l0.1,0.2h0.1
        l0.1,0.2l0.3,0.2L581.2,433.2L581.2,433.2z"/>
      <path id="TO" fill={selected == 'TO' ? selectedColor : hover == 'TO' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M13.3,707.7L13.3,707.7l-0.2,0.3v0.2l0.4,0.4L13.3,707.7z M11.7,706.8h-0.2H11.7l-0.4-0.3h-0.4l-0.2-0.1v-0.2
        l-0.2,0.3l0.2,0.3l0.9,0.4l0.3,0.2l0.2-0.6v-0.2l-0.3,0.1v0.1H11.7z M14.2,690.8l0.1-0.2v-0.2l-0.3-0.1h-0.1l-0.3,0.5l0.1,0.1
        l0.3,0.2h0.1L14.2,690.8z"/>
      <path id="TT" fill={selected == 'TT' ? selectedColor : hover == 'TT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M635.4,507.7l0.1-0.2v-0.6l0.2-0.4l-0.2-0.4l-0.1-0.6l0.1-0.5v-0.7l0.2-0.3l0.5-0.8h-0.9l-0.6,0.2l-1.1,0.1
        l-0.5,0.2l-0.7,0.1L632,504l0.1,0.1l0.5,0.2l0.2,0.2l0.1,0.2l0.1,0.4l-0.3,1.7l-0.1,0.1L632,507l-0.2,0.3l-1.4,0.8l0.8-0.1l0.9,0.1
        l2.4-0.1L635.4,507.7L635.4,507.7z M637.2,501l1.2-0.5l0.1-0.4h-0.2l-0.8,0.3l-0.6,0.5v0.2L637.2,501z"/>
      <path id="VC" fill={selected == 'VC' ? selectedColor : hover == 'VC' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M634.5,491.4L634.5,491.4v-0.1h0.1v-0.1l0,0v-0.1h-0.1v0.1l0,0v0.1h-0.1L634.5,491.4L634.5,491.4L634.5,491.4
        L634.5,491.4z M635.2,489.5l0.1-0.2l0.1-0.1l0,0l0,0l-0.1-0.1l0,0v0.1l-0.2,0.1l0,0v0.1l0,0v0.1H635h-0.1l0,0h0.1l0,0l0.1,0.1l0,0
        l0,0l0,0L635.2,489.5L635.2,489.5z M635.5,488.4l0.3-0.2l0.1-0.6l-0.1-0.4h-0.2l-0.3,0.1l-0.2,0.3l-0.1,0.5L635.5,488.4L635.5,488.4
        L635.5,488.4z"/>
      <path id="VG" fill={selected == 'VG' ? selectedColor : hover == 'VG' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M619.2,455.1l0.3-0.2l-0.2-0.1h-0.4l-0.3,0.2l0.1,0.1H619.2L619.2,455.1z M620.3,454.7l0.4-0.4l-0.5,0.1l-0.2,0.2
        l0.1,0.1h0.1L620.3,454.7L620.3,454.7z M621.1,452.9h-0.2h-0.5l0,0l0.1,0.1h0.3l0.3,0.1l0,0L621.1,452.9L621.1,452.9z"/>
      <path id="VI" fill={selected == 'VI' ? selectedColor : hover == 'VI' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M617.9,458.9l-0.7,0.2l-0.1,0.4h1.1l0.7-0.3h-0.6L617.9,458.9L617.9,458.9z M618.8,455.4l-0.5-0.1l-0.2,0.2l0,0
        l0.3,0.1L618.8,455.4z M617.7,455.5l-0.2-0.2l-0.3-0.1l-0.4,0.1l0.5,0.3L617.7,455.5L617.7,455.5z"/>
      <path id="CY" fill={selected == 'CY' ? selectedColor : hover == 'CY' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1149.9,348.4l-0.3-0.1l-0.5,0.2l-0.4,0.4l-0.4,0.3l-0.5-0.3l0.2,0.9l0.6,1.1l0.2,0.3l0.3,0.2l1.1,0.3h0.3h0.6
        l0.2,0.1l0.2,0.4h0.4v-0.1v-0.3l0.2-0.2l0.3-0.2h0.3l0.6-0.1l0.6-0.2l0.5-0.4l0.9-1h0.3h0.3h0.6l0.6-0.1l-0.2-0.4l-0.1-0.1l-0.4-0.5
        l-0.2-0.4l0.1-0.6l2.5-1.9l0.5-0.5l-0.8,0.2l-0.6,0.4l-0.4,0.2l-0.7,0.4l-2.3,0.8l-0.8,0.1h-0.8l-1-0.1l-0.9-0.2v0.7l-0.2,0.6
        l-0.6,0.2L1149.9,348.4L1149.9,348.4z"/>
      <path id="RE" fill={selected == 'RE' ? selectedColor : hover == 'RE' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1284,707.9l0.2-0.4l0.1-0.8l-0.4-0.8l-0.4-0.7l-0.4-0.2l-0.8-0.1l-0.7,0.3l-0.4,0.6l-0.2,0.3l0.4,1.1l0.2,0.3
        l1.1,0.6h0.5L1284,707.9L1284,707.9z"/>
      <path id="YT" fill={selected == 'YT' ? selectedColor : hover == 'YT' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1228.7,654.7v-0.3l0.2-0.5v-0.1l0.1-0.5l-0.3-0.3h-0.2l-0.2-0.3l-0.3,0.3l0.3,0.5l-0.1,0.3l-0.1,0.4l0.1,0.4
        l0.2,0.2L1228.7,654.7L1228.7,654.7z"/>
      <path id="MQ" fill={selected == 'MQ' ? selectedColor : hover == 'MQ' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M638,479.9l-0.2-0.7l-0.1-0.2l-0.2-0.3l0.1-0.3v-0.1h-0.2l-0.3-0.5l-0.6-0.3h-0.3l-0.2,0.2v0.3l0.3,0.9l0.2,0.2
        l0.5,0.2l-0.4,0.4v0.1l0.1,0.3h0.9l0.2,0.3l0.1-0.1L638,479.9L638,479.9z"/>
      <path id="GP" fill={selected == 'GP' ? selectedColor : hover == 'GP' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M636.4,471.1l0.2-0.2v-0.3l-0.2-0.3l-0.2,0.1l-0.2,0.3v0.3l0.1,0.1H636.4L636.4,471.1z M634.5,470.3l0.2-0.2v-1.2
        l0.1-0.3l-0.2-0.1l-0.2-0.2l-0.6-0.2l-0.1,0.1l-0.2,0.3l0.1,1.5l0.2,0.5l0.2,0.1L634.5,470.3L634.5,470.3z M636.1,468.9l0.8-0.2
        l-0.9-0.6l-0.2-0.4v-0.3l-0.4-0.3l-0.2,0.2l-0.1,0.3l0.1,0.5l-0.3,0.4l0.1,0.4l0.4,0.1L636.1,468.9L636.1,468.9z"/>
      <path id="CW" fill={selected == 'CW' ? selectedColor : hover == 'CW' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M595.9,494.9v-0.6l-0.9-0.4v0.3l0.1,0.2l0.3,0.1l0.1,0.2l-0.1,0.6l0.2,0.3L595.9,494.9L595.9,494.9z"/>
      <path id="BH" fill={selected == 'BH' ? selectedColor : hover == 'BH' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1253,408.3l0.7-3l-0.5-0.9l-1.6,1.2l0.6,0.9l-0.2,0.7L1253,408.3z"/>
      <path id="XK" fill={selected == 'XK' ? selectedColor : hover == 'XK' ? hoverColor : defaultColor} on:focus={doNothing} on:mouseover={handleMouseOver} on:mouseout={handleMouseOut} on:blur={doNothing} on:click={handleClick} d="M1080,299.8l1.2-0.5l0.5-2l0.9-0.4l0.8,0.9l1,0.4l0.8,1l0.8,0.3l1.1,1.1h0.8l-0.5,1.5l-0.5,0.7l0.2,0.5l-1.1,0.2l-2.9,1l-0.1,1.2h-0.7l-0.5-2.3l-1.3-0.6l-1.3-1.6L1080,299.8z"/>
      </svg>
  
  </div>
</body>
