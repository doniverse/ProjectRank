<script lang="ts">
    // @ts-ignore
    import Carousel from 'svelte-carousel'

	import { v4 as uuid } from "uuid";
	import Result from "./Result.svelte";
	import VerticalList from "./VerticalList.svelte";
	import { PaperPlaneRight } from 'phosphor-svelte';

    export let preparedData : any[];
    export let parameters: string[];
    export let projects: any[];
    let projectsCount = projects.length;
    
    let result : any[] = []
    projects.forEach((ele, idx)=>{
        result.push({
            id: uuid(),
            label: ele.name,
            data: [],
            total_score: 0
        })
    })


    let state : 'ranking' | 'result' = 'ranking';

    const showResult = () => {
        state = 'result'
        for(let x = 0; x < preparedData.length; x++){
            let parameterPower = (1 / (x + 1))
            for(let i = 0; i < projectsCount; i++){
                let score = (1000 / projectsCount) * (projectsCount - i) // probably should remove pp
                result = result.map((ele: any, idx: number) => {
                    if(ele.label == preparedData[x].rank[i].name){
                        let updatedData = [...ele.data, score]
                        return {...ele, data: updatedData, total_score: ele.total_score + (score * parameterPower)}
                    }
                    return ele
                })
            }
        }
        result.sort((a, b) => b.total_score - a.total_score)
        console.log(result)
    }

    
    let caresoule : any;
    let caresouleIndex: number = 0;
    
    const handlePageChange = (e: any) => {
                caresouleIndex = e.detail;
    }
</script>

<div class="flex-center w-full select-none my-12">
    {#if state == 'ranking'}
        <div class="max-w-[450px] w-full sm:min-w-[420px]">
            <Carousel
                bind:this={caresoule}
                on:pageChange={handlePageChange}
                infinite={false}
                class='relative'
            >
                {#each preparedData as selection, idx (selection.id)}   
                    <div id={selection.id} class="flex flex-col w-full">
                        <span class="p-2 text-2xl">
                            <h1>{selection.param}</h1>
                            <p class="text-xs">Place higher if it's the best according to this parameter.</p>
                        </span>
                        <div class="p-1">
                            <VerticalList bind:items={selection.rank} listName={`ranking-${selection.id}`}/>
                        </div>                    
                    </div> 
                {/each}
                        <div slot="prev" class="relative">
                            <button class="absolute -bottom-9 left-3 btn btn-sm" on:click={() => caresoule.goToPrev()} >Prev</button>
                        </div>

                        <div slot="next"  class="cursor-pointer">
                            {#if caresouleIndex >= (parameters.length - 1)}
                                <button class="absolute -bottom-9 right-3 btn btn-sm btn-primary" on:click={showResult} >
                                    Submit 
                                    <PaperPlaneRight size={18} color="#fcfcfc" />
                                </button>
                            {:else}
                                <button class="absolute -bottom-9 right-3 btn btn-sm" on:click={() => caresoule.goToNext()} >Next</button>
                            {/if}
                        </div>
                    <div slot="dots">
                        <!-- removes dots from caresoule -->
                    </div>
            </Carousel>
        </div>
    {:else if state == 'result'}
            <Result data={result} {parameters}/>
    {/if}
</div>


    


