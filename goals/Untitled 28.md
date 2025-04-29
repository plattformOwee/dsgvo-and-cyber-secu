For my Dating app profile page i want to be able to fleixbly control the layout for each person (order and content).
To make that possible, the entire layout should be driven by one json like:
{
	"profile_json":{
		"top_section":{ // will have same order and layout always
			"profile_images":{"linktoimage.com", "linktoimage.com", "linktoimage.com"}
			"name":"username"
			"age":"the age"
			"info_bubbles":{"searching":"relationship","live in":"live in ...", "school":"schule", "political leaning":"politik", "religions":"christ"},
		},
		"elements":{
			"1":{
				"type":"icebreaker" // a question on top and the answer below
				"content": {"question":"Meine Familie weiß immer noch nicht, dass ich...", "answer":"unsere weiße couch kaputt gemacht habe"}
			},
			"2":{
				"type":"image" // an image possibly with a subtitle
				"content": {"image_link":"example.com", "subtitle":"Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt."}
			}
			"3":{
				"type":"bubbles" // bubbles that should be sized optimally in grid showing something in this case, activities the person wants to do
				"content": {"title":"lets go", "bubbles":["skiing","chilling in nature","do random shit", "to a bar"]}
			}
			"4":{
				"type":"icebreaker" // a question on top and the answer below
				"content": {"question":"ich habe lust auf ...", "answer":"einfach mal chillen"}
			}
		}
	},
	
}
As you can see in the profile_json above, the first section will always look the same, so this part of the json will not have to be felxible and can be always the same hard built in build widget part.
But below, the "elements" will have a different order for each user, that means > this section must be built from the json by looking at the type and depending on the type, use a different element > each element is refering to a different seperate layout dart file which gets fed the "content"

The different elements are:
- icebreaker Q&A
- image and subtitle
- bubble grid

So to make all this possible:
1. create icebreaker Q&A icebreaker_layout.dart
	1. accepts json: "{"question":"the question here", "answer":"the answer here"}"
	2. displays question on top, the answer below
2. create image and subtitle image_layout.dart 
	1. accepts json: ""image_link":"example.com", "subtitle":"the subtitle""
	2. display image on top, subtitle below it
3. craete bubble_grid_layout.dart file
	1. accepts json: "{"title":"title of section", "bubbles":{"bubble example","bubble example","bubble example", "bubble example"}"
4. create profile.dart 
	1. eample json hardcoded on top in variable
	2. puts the data of the first section into this part of layout
	3. builds item list from the "elements" and use the according layout files