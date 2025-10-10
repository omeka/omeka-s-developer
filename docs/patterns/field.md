# Field

Fields are entries in a form. The field's main container is `<div class="field">`. If it is a required field, it is Labels and their descriptions are grouped in `<div class="field-meta">`, while associated inputs are in `<div class="inputs">`. 

![An example of a field, with "Email" as the label and a text input for entering the value.](../img/field.jpg)

```
<div class="field required" id="email_field">
    <div class="field-meta">
        <label for="email">Email</label>            
    </div>
    <div class="inputs">
        <input type="email" name="user-information[o:email]" id="email" required="" value="admin@omeka.org">    
    </div>
</div>
```