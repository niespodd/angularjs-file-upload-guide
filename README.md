# Painless file upload with AngularJS and Django REST

A common pattern when creating an application from scratch in Angular and Django is when there's a need for a form with any file upload. Let's say profile edition with avatar upload or an gallery with multiple file upload. Angular does not provide support for file upload by default. Developers over internet suggest many different solutions, but personally none of them cover the issue completely and painless.

### Endpoint setup

First, lets create sample Django REST application - model, serializer and view.

```python
# models.py
class Foo(models.Model):
    name = models.CharField(max_length=20)
    image = models.ImageField(upload_to=settings.IMAGES_PATH)
```

```python
# serializers.py
class FooSerializer(ModelSerializer):
    class Meta:
        model = Foo
```
```python        
# views.py
from rest_framework.parsers import MultiPartParser

class FooCreateView(CreateViewAPI):
    parser_classes = (MultiPartParser,)
    serializer_class = FooSerializer
```

This will simply provide an endpoint for uploading images with some extra ```name``` parameter to the server. As you can see ```MultipartParser``` is being used in the view additionaly.

> ```MultipartParser``` is a parser for multipart form data, which may include file data. It parses the incoming bytestream as a multipart encoded form, and returns a DataAndFiles object.

## Angular project setup

Second, we will create a form with a file input, controller and resource service.

```html
<form ng-controller="UploadFormController" ng-submit="submit(form)">
    <input type="file" ng-model="form.image" />
    <input type="text" ng-model="form.name" />
    <input type="submit" />
</form>
```
```coffeescript
angular.module 'application'
  .controller 'UploadFormController', (Profile) ->
    $scope.submit = (profile) ->
        Profile.update(profile).then (new_profile) ->
            console.log new_profile
```
```coffeescript
angular.module 'application'
  .service 'Profile', ($resource) ->
    
    @resource = $resource 'http://endpoint/url/', null
        update:
            method: 'POST'
            transformRequest: angular.identity
            headers:
                'Content-Type': undefined
                
    @update = (profile) ->
        formData = new FormData()
        
```