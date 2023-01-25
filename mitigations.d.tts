// This section allows for compatibility with the JQuery extension API.
// For type sanity, I recommend only allowing the getInstance method to be called this way.
interface JQuery
{
	// Provide access to the getInstance method
	jqxGrid(w: 'getInstance'): jqwidgets.jqxGrid;
	// ... Add more as needed.
	// If you need compatibility with a lot of existing stuff:
	jqxWhatever(w: any): void;
}

namespace jqwidgets
{
	interface GridColumn
	{
		// Needed because getting the columns property returns 
		// the wrong type as specified in their .d.ts file
		// To fix in your code, cast it with `as unknown as jqwidgets.GridColumn`
		records?: any[];
	}
	
	/** This overload exists because jqxTabs constructor function is broken */
	export function createInstance(selector: string, widget, 'jqxTabs', options: jqwidgets.TabsOptions)
		: jqwidgets.jqxTabs;
	// If you do not wish to use constructor syntax to create instances, add more overloads here.
		
	interface jqxLoader
	{
		// If a method/property is missing in the .d.ts, you can always add it here
		open(): void;
	}
	
	// Allow access to constructor syntax
	class jqxGrid implements jqwidgets.jqxGrid extends jqwidgets.widget {};
	// ... Add more as needed. Do NOT permit jqxTabs as it doesn't work.
}
