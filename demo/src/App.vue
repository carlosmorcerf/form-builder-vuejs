<template>
	<div id="app">
		<div class="container">
			<div class="row">
				<div class="col">
					<h2>Form Builder</h2>

					<FormBuilder :formId="'form-builder-container'" :containerStyles="'padding: 15px'" :initialSchema="initialSchema" :onCancel="cancelForm" :onSave="saveForm" />
				</div>
				<div class="col">
					<h2>Schema</h2>
					<div style="background: #fff; border: 1px solid #f2f2f2; padding: 15px; margin: 15px">
						<pre>{{ JSON.stringify(code, null, 2) }}</pre>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import FormBuilder from "../../src/components/FormBuilder";

const schema = {
	schemaVersion: "formbuilder-v1",
	fields: [
		{
			type: "text",
			label: "Nome",
			name: "nome",
			placeholder: "",
			defaultValue: "",
			tip: "",
			values: [],
			status: true,
			cssClasses:null,
			validations: {
				required: true,
				regexPattern: null,
				minlength: null,
				maxlength: 100,
			},
		},
		{
			type: "email",
			label: "E-mail",
			name: "email",
			placeholder: "",
			defaultValue: "",
			tip: "",
			values: [],
			status: true,
			cssClasses:"campo-email",
			validations: {
				required: true,
				regexPattern: null,
				minlength: 5,
				maxlength: 150,
			},
		},
		{
			type: "select",
			label: "Profissão",
			name: "profissao",
			placeholder: "",
			defaultValue: "",
			tip: "",
			status: true,
			cssClasses:"",
			values: [
				{ label: "Profissão 1", value: "1" },
				{ label: "Profissão 2", value: "2" },
				{ label: "Profissão 3", value: "3" },
			],
			validations: {
				required: false,
				regexPattern: null,
				minlength: null,
				maxlength: null,
			},
		},
	],
};

export default {
	name: "App",
	components: {
		FormBuilder,
	},
	methods: {
		cancelForm: function () {
			alert("Formulário cancelado!");
		},
		saveForm: function (success, schema, error) {
			if (success) {
				this.code = schema;
				//alert(JSON.stringify(schema));
			} else {
				if (error) {
					alert(error);
				} else {
					alert("Formulário inválido");
				}
			}
		},
	},
	data: function () {
		return {
			code: schema,
			initialSchema: schema,
		};
	},
};
</script>
