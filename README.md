See files in this repro as an example of implementation. Note: these behavior/resource packs are not complete so use these as a guide only for your specific implementation.

Add the papercraft example script either to your main.ts or as it's own file. if you add it as it's own file, make sure to add an import the const "initExampleStickers" in your main. This structure is strict though you can add as many stickers as you want, you may need to adjust the payload size, section 4.

```
// 1) Make sure you import Entity, system and world from server:
import { Entity, system, world } from "@minecraft/server";

// 2) Replace the examples with the entities that you want to add to pcs and copy/paste it to your main file
// Use the minecraft id as the keys of the object. The contents is an array of objects, if they have conditions, they are evaluated sequentially.
// The default one (no conditions) is used if no condition match
// - (required) itemId - Id of the sticker (item)
// - (optional) specialItemId - Id of the special sticker (item). Harder to obtain!
// - (optional)  conditions - Array If the same Entity could transform on different items depending on a supported component value or property
//                            All the conditions must match
// Conditions can be a component:
//  - component: One of the supported components: minecraft:color, minecraft:variant, minecraft:mark_variant or minecraft:is_sheared
//  - componentValue: a number to match against the value of the component. This value is ignored for `minecraft:is_sheared`
// or a property:
// - property: name of the entity property
// - propertyValue: Value of the property
export const initExampleStickers = () => {
  const entities: { [entityType: string]: Array<{ conditions?: Array<{ component: string; componentValue?: number; } | { property: string; propertyValue: ReturnType<Entity['getProperty']>; }> } & ({ itemId: string; specialItemId?: string; })> } = {
    'your_namespace:example_mob': [{itemId: 'your_namespace:normal_sticker',specialItemId: 'your_namespace:rare_sticker'}],
    'your_namespace:example_complex_mob': [
		{itemId: 'your_namespace:normal_sticker_1',specialItemId: 'your_namespace:rare_sticker_1',conditions: [{component: 'minecraft:variant',componentValue: 0}]},
		{itemId: 'your_namespace:normal_sticker_2',specialItemId: 'your_namespace:rare_sticker_2',conditions: [{component: 'minecraft:variant',componentValue: 1}]}	],
  };

  // 3) Data Payload Construction
const chunkObject = (obj: typeof entities, chunkSize: number) => {
	const keys = Object.keys(obj);
	const chunks: Array<typeof entities> = [];
	for (let i = 0; i < keys.length; i += chunkSize) {
	  const chunk: typeof entities = {};
	  keys.slice(i, i + chunkSize).forEach(key => {
		chunk[key] = obj[key];
	  });
	  chunks.push(chunk);
	}
	return chunks;
  };
  
  // 4) If the payload is too large, reduce number here to smaller chunks of data. only necessary if there are many variants on single entities.
  const entityChunks = chunkObject(entities, 4);
  
   entityChunks.forEach((entityChunk, index) => {
	const m = {
	  id: (Math.random()).toString(16),
	  type: 'request',
	  endpoint: 'addStickers',
	  params: [1, entityChunk], // Use only a chunk of entities
	  callback: undefined,
	};
  
	const delay = 10 + index; // 1 tick additional delay for each chunk
  
	system.runTimeout(
	  () =>
		void world
		  .getDimension("overworld")
		  .runCommandAsync(`scriptevent mineapi:jig_pcs ${JSON.stringify(m)}`),
	  delay
	);
  
  });
  
  // Debug: Initialization complete
}
```
