# AEM Dialog Field Types Reference

Complete reference for Granite UI dialog field types with examples.

## Text Fields

### Textfield
```xml
<title jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Title"
    name="./title"
    required="{Boolean}true"
    maxlength="80"
    emptyText="Enter a title"/>
```

### Textarea
```xml
<description jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
    fieldLabel="Description"
    name="./description"
    rows="5"
    maxlength="500"/>
```

### Rich Text Editor
```xml
<body jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/authoring/dialog/richtext"
    fieldLabel="Body Content"
    name="./body"
    useFixedInlineToolbar="{Boolean}true">
    <rtePlugins jcr:primaryType="nt:unstructured">
        <format jcr:primaryType="nt:unstructured"
            features="bold,italic"/>
        <links jcr:primaryType="nt:unstructured"
            features="modifylink,unlink"/>
        <lists jcr:primaryType="nt:unstructured"
            features="unordered,ordered"/>
    </rtePlugins>
    <uiSettings jcr:primaryType="nt:unstructured">
        <cui jcr:primaryType="nt:unstructured">
            <inline jcr:primaryType="nt:unstructured"
                toolbar="[format#bold,format#italic,-,links#modifylink,links#unlink,-,lists#unordered,lists#ordered]"/>
        </cui>
    </uiSettings>
</body>
```

## Selection Fields

### Select/Dropdown
```xml
<layout jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/select"
    fieldLabel="Layout"
    name="./layout">
    <items jcr:primaryType="nt:unstructured">
        <default jcr:primaryType="nt:unstructured"
            text="Default"
            value="default"
            selected="{Boolean}true"/>
        <wide jcr:primaryType="nt:unstructured"
            text="Wide"
            value="wide"/>
        <compact jcr:primaryType="nt:unstructured"
            text="Compact"
            value="compact"/>
    </items>
</layout>
```

### Checkbox
```xml
<hideTitle jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/checkbox"
    text="Hide Title"
    name="./hideTitle"
    value="{Boolean}true"
    uncheckedValue="{Boolean}false"/>
```

### Radio Group
```xml
<alignment jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/radiogroup"
    fieldLabel="Alignment"
    name="./alignment"
    vertical="{Boolean}true">
    <items jcr:primaryType="nt:unstructured">
        <left jcr:primaryType="nt:unstructured"
            text="Left"
            value="left"
            checked="{Boolean}true"/>
        <center jcr:primaryType="nt:unstructured"
            text="Center"
            value="center"/>
        <right jcr:primaryType="nt:unstructured"
            text="Right"
            value="right"/>
    </items>
</alignment>
```

### Switch/Toggle
```xml
<enabled jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/switch"
    fieldLabel="Enable Feature"
    name="./enabled"
    value="{Boolean}true"
    uncheckedValue="{Boolean}false"
    checked="{Boolean}true"/>
```

## Path and Reference Fields

### Path Browser
```xml
<link jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
    fieldLabel="Link"
    name="./link"
    rootPath="/content"
    filter="hierarchyNotFile"/>
```

### Content Fragment Reference
```xml
<fragmentPath jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
    fieldLabel="Content Fragment"
    name="./fragmentPath"
    rootPath="/content/dam"
    filter="hierarchy"
    droppable="{Boolean}true"/>
```

### Tag Field
```xml
<tags jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/coral/common/form/tagfield"
    fieldLabel="Tags"
    name="./tags"
    multiple="{Boolean}true"
    rootPath="/content/cq:tags/myproject"/>
```

## Media Fields

### File Upload (Image)
```xml
<file jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/authoring/dialog/fileupload"
    fieldLabel="Image"
    name="./file"
    fileNameParameter="./fileName"
    fileReferenceParameter="./fileReference"
    allowUpload="{Boolean}true"
    mimeTypes="[image/gif,image/jpeg,image/png,image/webp]"
    uploadUrl="${suffix.path}"
    useHTML5="{Boolean}true"/>
```

### Image Component (with cropping)
```xml
<image jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/authoring/dialog/fileupload"
    fieldLabel="Image"
    name="./image/file"
    fileNameParameter="./image/fileName"
    fileReferenceParameter="./image/fileReference"
    allowUpload="{Boolean}true"
    class="cq-droptarget"
    title="Drop an image">
    <granite:rendercondition jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/renderconditions/simple"
        expression="${empty cqDesign.disableFileUpload}"/>
</image>
```

## Date and Number Fields

### Date Picker
```xml
<publishDate jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/datepicker"
    fieldLabel="Publish Date"
    name="./publishDate"
    type="datetime"
    displayedFormat="YYYY-MM-DD HH:mm"/>
```

### Number Field
```xml
<priority jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/numberfield"
    fieldLabel="Priority"
    name="./priority"
    min="1"
    max="10"
    step="1"
    value="5"/>
```

## Multifield Patterns

### Simple Multifield
```xml
<links jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
    fieldLabel="Links">
    <field jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
        name="./links"/>
</links>
```

### Composite Multifield (Recommended for AEM 6.5+)
```xml
<cards jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
    composite="{Boolean}true"
    fieldLabel="Cards">
    <field jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container"
        name="./cards">
        <items jcr:primaryType="nt:unstructured">
            <title jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                fieldLabel="Title"
                name="./title"
                required="{Boolean}true"/>
            <description jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                fieldLabel="Description"
                name="./description"/>
            <image jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                fieldLabel="Image"
                name="./image"
                rootPath="/content/dam"/>
            <link jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                fieldLabel="Link"
                name="./link"
                rootPath="/content"/>
        </items>
    </field>
</cards>
```

## Layout Components

### Tabs Container
```xml
<content jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/container">
    <items jcr:primaryType="nt:unstructured">
        <tabs jcr:primaryType="nt:unstructured"
            sling:resourceType="granite/ui/components/coral/foundation/tabs"
            maximized="{Boolean}true">
            <items jcr:primaryType="nt:unstructured">
                <general jcr:primaryType="nt:unstructured"
                    jcr:title="General"
                    sling:resourceType="granite/ui/components/coral/foundation/container">
                    <items jcr:primaryType="nt:unstructured">
                        <!-- General tab fields -->
                    </items>
                </general>
                <advanced jcr:primaryType="nt:unstructured"
                    jcr:title="Advanced"
                    sling:resourceType="granite/ui/components/coral/foundation/container">
                    <items jcr:primaryType="nt:unstructured">
                        <!-- Advanced tab fields -->
                    </items>
                </advanced>
            </items>
        </tabs>
    </items>
</content>
```

### Accordion
```xml
<accordion jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/accordion"
    variant="default">
    <items jcr:primaryType="nt:unstructured">
        <section1 jcr:primaryType="nt:unstructured"
            jcr:title="Section 1"
            sling:resourceType="granite/ui/components/coral/foundation/container">
            <items jcr:primaryType="nt:unstructured">
                <!-- Section 1 fields -->
            </items>
        </section1>
    </items>
</accordion>
```

### Fieldset
```xml
<styling jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/fieldset"
    jcr:title="Styling Options">
    <items jcr:primaryType="nt:unstructured">
        <!-- Grouped fields -->
    </items>
</styling>
```

## Conditional Rendering

### Show/Hide with granite:class
```xml
<advancedOptions jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/checkbox"
    text="Show Advanced Options"
    name="./showAdvanced"
    value="true"
    granite:class="cq-dialog-checkbox-showhide"
    cq-dialog-checkbox-showhide-target=".advanced-options"/>

<advancedContainer jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/container"
    granite:class="advanced-options hide">
    <!-- Advanced fields that show/hide -->
</advancedContainer>
```

### Render Condition
```xml
<authorOnly jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Author Note"
    name="./authorNote">
    <granite:rendercondition jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/renderconditions/simple"
        expression="${granite:contains(granite:requestContext(), 'author')}"/>
</authorOnly>
```

## Validation

### Required Field
```xml
<title jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Title"
    name="./title"
    required="{Boolean}true"
    validation="foundation.jcr.name"/>
```

### Pattern Validation
```xml
<email jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Email"
    name="./email"
    validation="email"/>

<url jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="URL"
    name="./url"
    validation="url"/>
```

## Data Sources

### Static Options from Data Source
```xml
<country jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/select"
    fieldLabel="Country"
    name="./country">
    <datasource jcr:primaryType="nt:unstructured"
        sling:resourceType="myproject/datasources/countries"/>
</country>
```

### Tag-based Options
```xml
<category jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/select"
    fieldLabel="Category"
    name="./category">
    <datasource jcr:primaryType="nt:unstructured"
        sling:resourceType="cq/gui/components/common/datasources/tags"
        tagRootPath="/content/cq:tags/myproject/categories"/>
</category>
```
