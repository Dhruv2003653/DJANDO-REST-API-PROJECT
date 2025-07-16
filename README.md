HELLO, Project from DHRUV AHIR.
Lets consider "INDIA" as the name of our project and "MAHA" as the name of our App.

##for creation of Project and App
django-admin startproject INDIA
cd INDIA
python manage.py startapp MAHA
pip install djangorestframework


##for MODELS (MAHA/models.py)
from django.db import models
from django.contrib.auth.models import User

class Client(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()
    hobbies = models.TextField()
    college_name = models.CharField(max_length=200)

    def __str__(self):
        return self.name

class Project(models.Model):
    name = models.CharField(max_length=100)
    client = models.ForeignKey(Client, on_delete=models.CASCADE, related_name='projects')
    users = models.ManyToManyField(User, related_name='projects')

    def __str__(self):
        return self.name


##in order to let the project know about our app (INDIA/settings.py)
INSTALLED_APPS = [
    ...
    'rest_framework,
    'MAHA',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
]


##lets now introduce "Serializers" (MAHA/serializers.py)
from rest_framework import serializers
from .models import Client, Project
from django.contrib.auth.models import User

class ClientSerializer(serializers.ModelSerializer):
    class Meta:
        model = Client
        fields = '__all__'

class ProjectSerializer(serializers.ModelSerializer):
    class Meta:
        model = Project
        fields = '__all__'

class ProjectDetailSerializer(serializers.ModelSerializer):
    client = ClientSerializer()
    users = serializers.StringRelatedField(many=True)

    class Meta:
        model = Project
        fields = '__all__'


##lets visit views of our project (MAHA/views.py)
from rest_framework import viewsets, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from .models import Client, Project
from .serializers import ClientSerializer, ProjectSerializer, ProjectDetailSerializer

class ClientViewSet(viewsets.ModelViewSet):
    queryset = Client.objects.all()
    serializer_class = ClientSerializer
    permission_classes = [permissions.IsAuthenticated]

class ProjectViewSet(viewsets.ModelViewSet):
    queryset = Project.objects.all()
    permission_classes = [permissions.IsAuthenticated]

    def get_serializer_class(self):
        if self.action in ['list', 'retrieve', 'my_projects']:
            return ProjectDetailSerializer
        return ProjectSerializer

    @action(detail=False, methods=['get'])
    def my_projects(self, request):
        user = request.user
        projects = user.projects.all()
        serializer = ProjectDetailSerializer(projects, many=True)
        return Response(serializer.data)


##in order to avoid error we also have to make slight changes into URLS of our PROJECT as well as of APP
## for app (MAHA/urls.py)
from rest_framework.routers import DefaultRouter
from .views import ClientViewSet, ProjectViewSet
from django.urls import path, include

router = DefaultRouter()
router.register('clients', ClientViewSet)
router.register('projects', ProjectViewSet)

urlpatterns = [
    path('', include(router.urls)),
]


## now,  for project(INDIA/urls.py)
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('MAHA.urls')),
]


##to Run Migrations and Create Superuser
python manage.py makemigrations MAHA
python manage.py migrate
python manage.py createsuperuser



## for Creating Clients
POST /api/clients/ with:
for client named "DHRUV"
{
  "name": "Dhruv",
  "age": 25,
  "hobbies": "Cricket, Coding",
  "college_name": "M.H SABOO SIDDIK"
}
## for Fetching Clients
GET /api/clients/
##if you want to edit
PUT /api/clients/<id>/
##if u want to delete
DELETE /api/clients/<id>/

##to Add Project and Assign Users
{
  "name": "Smart India Hackathon",
  "client": 1,
  "users": [1, 2]
}

##to Get Projects Assigned to Logged-in User
GET /api/projects/my_projects/

## finally we have finished the task. 
