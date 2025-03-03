# Unofficial Mineplex Studios Bedrock Compatibility Resource Pack

This is a resource pack I created for the purpose of allowing multiple studio partners to be able to mess around with common JSON UI elements due to Bedrock's limitation of only one pack being able to change the implementation of the base JSON UI element implementation. By no means do I claim to be a JSON UI wizard and I welcome feedback/additions/fixes to this pack!

It supports modification of...

- Server Forms
- Scoreboards
- Titles
- Subtitles
- Actionbars

There are also built-in utility modifications that represent common uses for a resource pack. Check them out before you decide if you need to create a resource pack yourself. All studios are able to use these utility modifications!

**I recommend that you experiment with JSON UI locally before you attempt to create your own custom UI with this pack. This pack only provides a way for multiple studios to extend the same UI element. It does NOT handle your UI element failing to hide when it is meant to hide/extending unsupported UI elements. It is EXPECTED that you know how Bedrock resource packs work already.**

## How do I extend one of the UIs above?

The key to this resource pack is using the `modifications` property in your own RP's JSON files. Let's say that you want to create a unique server form UI.

Your resource pack file structure should look something like this:

```
manifest.json
ui/
 mineplex/
  server_form.jsonc
```

In your `manifest.json`, make sure that you add the compatibility resource pack to your dependencies.
```json
"dependencies": [
    {   // This is the UUID of the Compatibility Resource Pack!
        "uuid": "e7d40767-2588-4525-ba06-bac7057857a5",
        "version": [1, 0, 0]
    }
]
```

In `ui/mineplex/server_form.jsonc`, you will be extending the existing `mineplex_server_form.form` element to add an additional element to its controls.
```json
{
    "namespace": "mineplex_server_form",

    "form": {
        "modifications": [
            {
                "array_name": "controls",
                "operation": "insert_back",
                "value": {
                    // As an example, assume that this has all the same properties as 
                    // the original Vanilla long_form@common_dialogs.main_panel_no_buttons
                    // In your case, you will probably want to customize this form!
                    "my_custom_form@common_dialogs.main_panel_no_buttons": {
                        // ......

                        // We want to ONLY show this custom form IF the title of the form starts with com.mineplex.studio.neuv.my_custom_form
                        "bindings": [
                            {
                                "binding_name": "#title_text"
                            },
                            {
                                "source_property_name": "(not (#title_text = (#title_text - 'com.mineplex.studio.neuv.my_custom_form')))",
                                "binding_type": "view",
                                "target_property_name": "#visible"
                            }
                        ],

                        // We need to specify this as well as we need to hide our namespace from the displayed title
                        "controls": [
                            {
                                "common_panel@common.common_panel": { "$dialog_background": "$custom_background" }
                            },
                            {
                                "title_label@common_dialogs.title_label": {
                                    "controls": [
                                        {
                                            "common_dialogs_0@standard_title_label": {
                                                "text": "#title_text",
                                                "bindings": [
                                                    {
                                                        "binding_name": "#title_text"
                                                    },

                                                    // Remove our namespace shown to the player
                                                    {
                                                        "source_property_name": "(#title_text - 'com.mineplex.studio.neuv.my_custom_form')",
                                                        "binding_type": "view",
                                                        "target_property_name": "#title_text"
                                                    }
                                                ]
                                            }
                                        }
                                        
                                    ]
                                }
                            },
                            {
                                "panel_indent": {
                                    "type": "panel",
                                    "size": "$panel_indent_size",
                                    "offset": [ 0, 23 ],
                                    "anchor_from": "top_middle",
                                    "anchor_to": "top_middle",
                                    "controls": [
                                        { "inside_header_panel@$child_control": {} }
                                    ]
                                }
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

On your Bedrock Client, make sure that you have your resource pack enabled. 

Now, whenever a server form appears with a title that starts with `com.mineplex.studio.neuv.my_custom_form`, our custom server form will be shown to the player.

Similar logic can be followed for the other related elements. The key point is to only show your element if your prefix is found in the UI element you are modifying. Check out `resource-pack/ui/mineplex/builtin` for some more examples on how to know when to hide/show your element.

## How does it work?

This pack overrides the Vanilla implementation of the UI element and replaces it with a wrapper around the original implementation. The copied version will NOT show the original Vanilla UI if the identifier for the element begins with `com.mineplex.studio.` . 

Studios can then use the `modifications` property to add additional UI elements to the wrapper of the target UI element, and these implementations can then be specified to only appear if the identifier begins with their namespace. (e.g. `com.mineplex.studio.neuv.my_custom_form`)

## Credit

I want to say thank you to [Dingsel](https://www.youtube.com/@Dingsel) for provided helpful JSON UI tutorials that became incredibly useful while developing the built-in image grid and icon grid server forms.