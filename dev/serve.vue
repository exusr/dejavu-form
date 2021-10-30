<template>
	<div id="app">
		<dejavu-form :content="this.form"/>
	</div>
</template>

<script>
import { defineComponent } from 'vue';

export default defineComponent({
	name: 'ServeDev',
	data() {
		return {
			form: {
				action : "http://localhost:8080/post",
				method : "POST",
                type : "multipart/form-data",
                mode : "cors",
                cache : "default",
                credentials : "omit",
				attributes : {
					id: "dejavu-form", class: "dejavu-form", //style: "border: 1px solid green;"
				},
				inputs: [
					{
						id: "text", type: "text", label: "text", value: "text", placeholder: "text", name: "text",
						attributes: {
							id: "text-id",
						},
						validation: {
							required: true,
							match: /^dejavu$/
						},
                        errors: {
                            "required" : "overwrite required error message",
                            "match" : "overwrite match message",
                        }
					}, {
						id: "select", type: "select", label: "select", name: "select",
						options: [
							{ value: 0, display: "0", selected: false },
							{ value: 1, display: "1", selected: true },
						],
						validation: {
							required : false,
						}
					}, {
						id: "number", type: "number", label: "number", name: "number", step: "0.001", value:10,
						validation: {
							required : false,
							min : 3,
							max : 10
						}
					}, {
						id: "file", type: "file", label: "file", name: "file", mutliple: true,
						validation : {
                            min: "10kb",
							required : false,
							extensions : ["png"],
                            max: "500kb"
						}
					}, {
						id: "password", type: "password", label: "password", name: "password", disabled: true
					}, { 
						id: "checkbox", type: "checkbox", label: "checkbox", name: "checkbox", checked: true, value:1, display:"checkbox"
					}, {
						id: "color", type: "color", label: "color", name: "color"
					}, {
						id: "date", type: "date", label: "date", name: "date",
						validation : {
							required : false,
							min : new Date(2021, 12, 30),
							max : new Date(2022, 12, 30),
						}
					}, {
						id: "datetime-local", type: "datetime-local", label: "datetime-local", name: "datetime-local",
						validation : {
							required : false,
							min : new Date(2021, 12, 30, 21, 21, 21),
							max : new Date(2022, 12, 30, 22, 22, 22)
						}
					}, {
						id: "email", type: "email", label: "email", name: "email"
					}, {
						id: "hidden", type: "hidden", label: "hidden", name: "hidden"
					}, {
						id: "month", type: "month", label: "month", name: "month"
					}, {
						id: "radio", type: "radio", label: "radio", name: "radio", value:1, display:"radio"
					}, {
						id: "range", type: "range", label: "range", name: "range"
					}, {
						id: "reset", type: "reset", label: "reset", name: "reset"
					}, {
						id: "search", type: "search", label: "search", name: "search"
					}, {
						id: "tel", type: "tel", label: "tel", name: "tel"
					}, {
						id: "text", type: "text", label: "text", name: "text"
					}, {
						id: "time", type: "time", label: "time", name: "time"
					}, {
						id: "url", type: "url", label: "url", name: "url"
					}, {
						id: "week", type: "week", label: "week", name: "week"
					}, {
						id: "textarea", type: "textarea", label: "textarea", name: "textarea",
						row: 10, cols: 30, required: false, value: "Lorem ipsum dolor sit amet",
						validation : {
							max: 200,
							min: 2
						}
					}, {
						id: "submit", type: "submit", label: "submit", name: "submit"
					}
				]
			}
		}
	},
    methods: {
        async formCallback(response) {
            alert( ( await response.json() ).test );
        }
    }
});
</script>

<style>
	html {
		background: #292d3e;
	}
</style>
