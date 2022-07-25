<script>
	let frequency = 10
	let allMicLabels = 'ABCD'.split('')
	let allSourceLabels = 'ABCD'.split('')
	let allMicrophonePositions = Array(allMicLabels.length).fill(null).map((_,i, all) => -250 + 500 * (i)/(all.length-1))
	let allSources = Array(allSourceLabels.length).fill(null).map((_,i, all) => Math.PI * (i+1)/(all.length+1))
	let sourceColors = Array(allSourceLabels.length).fill(null).map((_,i, all) => `hsl(${Math.round(360 * ((i+1)*3)/all.length+1)},90%,40%)`)
	let sourceVisiblity = Array(allSourceLabels.length).fill(true)
	let sourceType = Array(allSourceLabels.length).fill(true).map((_,i) => i==0)
	let numberOfMicrophones = 3
	let numberOfSources = 3
	$: targetAngles = allSources.slice(0, numberOfSources)
	$: microphonePositions = allMicrophonePositions.slice(0, numberOfMicrophones).sort((a,b) => (a-b))
	$: micLabels = Array(numberOfMicrophones).fill(null).map((_,i) => i).sort((a,b) => (allMicrophonePositions[a]-allMicrophonePositions[b])).map((i) => allMicLabels[i])
	
	
	$: waveDirections = targetAngles.map((targetAngle) => {
		const targetX = -Math.cos(targetAngle) * 500
		const targetY = -Math.sin(targetAngle) * 500
		
		return microphonePositions.map((mX) => {
			const waveX = targetX - mX
			const waveY = targetY - 0

			const targetLength = Math.sqrt(targetX*targetX + targetY*targetY)
			const waveLength = Math.sqrt(waveX*waveX + waveY*waveY)

			const dot = (waveX*targetX + waveY*targetY) / targetLength

			return [targetX/targetLength*dot, targetY/targetLength*dot, dot, microphonePositions[targetAngle > Math.PI/2 ? 0 : microphonePositions.length-1] - mX]
		})
	})
</script>

<style>
	input[type=range] {
		max-width: 10em;
	}
	
	label, dt {
		white-space: nowrap;
	}
	
	.radio-label {
		display: flex;
		gap: 0.5em;
		align-items: baseline;
		margin: 0 0.5em;
	}

	article {
		max-width: 100ch;
		margin: auto;
	}
</style>
<article>
	

<h1>
	Beam Former
</h1>

<p>
	Below you can explore recording multiple sound sources with multiple microphones results yields a linear transformation in the spectral domain. If there are at least as many microphones as sound sources the linear transformation can be inverted to recover the individual source signals. A common case is to select one source as desired signal and assume the other sources as noise. Then one can reduce the noise by reverting the transformation and then selecting the clean source.
</p>

<p>
This tool is not yet fully implemented.
</p>

{#if numberOfSources > numberOfMicrophones}
<p style="background: #fcc;padding: 1em; color:#500;">
	Number of Microphones must be greater or equal to the number of sources. Otherwise the resulting linear transformation can not be inverted.
</p>
{/if}
 
<div style="display: grid; grid-template-columns: 1fr 1fr;">
<dl style="display: grid; grid-template-columns: auto 1fr;">
		<dt>Number of Microphones</dt>

	<dd><input type="number" min="1" max="{allMicLabels.length}" bind:value={numberOfMicrophones} /></dd>
	<dt>Number of Sources</dt>
	<dd><input type="number" min="1" max="{allMicLabels.length}" bind:value={numberOfSources} /></dd>
	<dt>Wavelength</dt>
	<dd><input type="range" min="5" step="5" max="{200}" bind:value={frequency} /></dd>
</dl>


	<dl style="display: grid; grid-template-columns: auto 1fr;align-content: start;">
	{#each microphonePositions as p,m}
	<dt>Microphone {allMicLabels[m]}</dt>
	<dd><input type="range" min="-250" max="250" bind:value={allMicrophonePositions[m]} /></dd>
	{/each}
</dl>

<dl style="display: grid; grid-template-columns: auto auto auto auto;align-content: start; justify-content: start;">
	{#each targetAngles as p,s}
	<dt style="color:{sourceColors[s]}">Source {allSourceLabels[s]}</dt>
	<dd><input type="checkbox" bind:checked={sourceVisiblity[s]} /></dd>
	<dd><input type="range" min={Math.PI*0.02} max={Math.PI*(1-0.02)} step={Math.PI/500} bind:value={allSources[s]} /></dd>
	<dd style="display: flex;">
		<label class="radio-label"><input type="radio" bind:group={sourceType[s]} value={true} /> Signal</label>
		<label class="radio-label"><input type="radio" bind:group={sourceType[s]} value={false} /> Noise</label>
	</dd>

	{/each}
</dl>
	<div style="display: grid; align-content: center; justify-content: center;">
	
		<math xmlns = "http://www.w3.org/1998/Math/MathML" >
<mrow>
	 <mo>[</mo>
      <mtable>
				{#each microphonePositions as m,j}
         <mtr>
            <mtd><msub>  
      <mi>M</mi>  
      <mn>{allMicLabels[j]}</mn>  
   </msub></mtd>
         </mtr>
	{/each}
         
      </mtable>
      <mo>]</mo>
		<mo>=</mo>
	<mo>[</mo>
      <mtable>
				{#each targetAngles as t,i}
         <mtr>
					 {#each microphonePositions as m,j}
					 
            <mtd><msub>  
      <mi>a</mi>  
      <mn>{i},{j}</mn>  
   </msub></mtd>
					 {/each}
         </mtr>
	{/each}
         
      </mtable>
      <mo>]</mo>
	
		<mo>&times;</mo>
	
      <mo>[</mo>
      <mtable>
				{#each targetAngles as t,i}
         <mtr>
            <mtd><msub>  
      <mi>S</mi>  
      <mn>{allSourceLabels[i]}</mn>  
   </msub></mtd>
         </mtr>
	{/each}
         
      </mtable>
      <mo>]</mo>
   </mrow>
</math>
</div>
</div>

<svg viewBox="-600 -600 1200 700">
	<path d="M-350,0H350" stroke="black" stroke-width="2" />
	<circle r="500" cx={0} cy={0} fill="none" stroke="gray" stroke-dasharray="20 50" /> 

	{#each waveDirections as w, i (i)}
	{#if sourceVisiblity[i]}

		{#each microphonePositions as p, j (j)}
	
	<path d="M{p},{0} l{w[j][0]},{w[j][1]}" stroke={sourceColors[i]} />
	<path d="M{p+w[j][0]},{w[j][1]}L{p},{0} " stroke={sourceColors[i]} stroke-width="5" stroke-dasharray="{frequency} {frequency}" />
	<path d="M{p+w[j][0]},{w[j][1]}L{p},{0} " stroke={sourceColors[i]} stroke-width="5" stroke-dasharray="{frequency} {frequency}" stroke-dashoffset="{frequency}" opacity="0.2" />
	<path d="M{p},{0} l{-w[j][1]/w[j][2]*w[j][3]*Math.sin(targetAngles[i])},{w[j][0]/w[j][2]*w[j][3]*Math.sin(targetAngles[i])}" stroke={sourceColors[i]} stroke-dasharray="3 3" />
	{/each}

		<path d="M{microphonePositions[0]+w[0][0]},{w[0][1]} L{microphonePositions[w.length-1]+w[w.length-1][0]},{w[w.length-1][1]}" stroke={sourceColors[i]} stroke-width="4" />
	
	
	<circle r="50" cx={(microphonePositions[0]+w[0][0]+microphonePositions[w.length-1]+w[w.length-1][0])/2 + -Math.cos(targetAngles[i]) * 40} cy={(w[0][1]+w[w.length-1][1])/2 -Math.sin(targetAngles[i]) * 40} fill={sourceColors[i]} opacity="0.1" /> 
	<circle r="5" cx={(microphonePositions[0]+w[0][0]+microphonePositions[w.length-1]+w[w.length-1][0])/2 + -Math.cos(targetAngles[i]) * 40} cy={(w[0][1]+w[w.length-1][1])/2 -Math.sin(targetAngles[i]) * 40} fill={sourceColors[i]} /> 
	
	<text fill={sourceColors[i]} x={(microphonePositions[0]+w[0][0]+microphonePositions[w.length-1]+w[w.length-1][0])/2 + -Math.cos(targetAngles[i]) * 40} y={(w[0][1]+w[w.length-1][1])/2 -Math.sin(targetAngles[i]) * 40 -15} text-anchor="middle">Source {allSourceLabels[i]}</text>

			{/if}
	{/each}
	
	
	{#each microphonePositions as p, i (i)}
	<circle r="20" cx={p} cy={0} fill="white" stroke="black" stroke-width="4" />
	<text x={p} y={45+ (i%2)*30} text-anchor="middle">Microphone {micLabels[i]}</text>
	{/each}
</svg>

<p style="text-align: center;">
	<a href="https://tools.laszlokorte.de/" title="more educational tools">more educational tools</a>
</p>

</article>