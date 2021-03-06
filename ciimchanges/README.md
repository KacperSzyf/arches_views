# Installation 

1. In `urls.py` add the following url:
  At the top of `urls.py` add `from .views.ciimchanges import ChangesView`
  and in `urlpatterns` add `url(r"^resource/changes", ChangesView.as_view(), name="ChangesView"),`
 
2. Add the `ciimchanges.py` file to your views folder

3. Add `LatestResourceEdit` model to `arches/app/models/models.py` just under `EditLog`:
    ```
    class LatestResourceEdit(models.Model):
      editlogid = models.UUIDField(primary_key=True, default=uuid.uuid1)
      username = models.TextField(blank=True, null=True)
      resourcedisplayname = models.TextField(blank=True, null=True)
      resourceinstanceid = models.TextField(blank=True, null=True)
      edittype = models.TextField(blank=True, null=True)
      timestamp = models.DateTimeField(blank=True, null=True)

    class Meta:
        managed = True
        db_table = "latest_resource_edit"
    ```
  
4. Add catch to `arches/app/models/resource.py` at the end of the `save_edit` function:
    ```
        if LatestResourceEdit.objects.filter(resourceinstanceid=self.resourceinstanceid, edittype = 'create').exists():
            if LatestResourceEdit.objects.filter(resourceinstanceid = self.resourceinstanceid).exclude(edittype = 'create').exists():
                LatestResourceEdit.objects.filter(resourceinstanceid = self.resourceinstanceid).exclude(edittype = 'create').delete()
            #Delete old verions and add latest edit
            latest_edit = LatestResourceEdit()
            latest_edit.resourceinstanceid = self.resourceinstanceid
            latest_edit.timestamp = timestamp
            latest_edit.username = getattr(user, "username", "")
            latest_edit.edittype = edit_type
            latest_edit.resourcedisplayname =  Resource.objects.get(resourceinstanceid=self.resourceinstanceid).displayname
            latest_edit.save()

        else:
            latest_edit = LatestResourceEdit()
            latest_edit.resourceinstanceid = self.resourceinstanceid
            latest_edit.timestamp = timestamp
            latest_edit.edittype = edit_type
            latest_edit.username = getattr(user,"username", "")
            latest_edit.resourcedisplayname =  Resource.objects.get(resourceinstanceid=self.resourceinstanceid).displayname
            latest_edit.save()
    ```
    
5. Run `python manage.py makemigrations` command

6. Run `python manage.py migrate` command

7. Add `populate_latest_resource_edit_table` to `your_project/your_project/managment/commands/`

8. Finally run `python manage populate_latest_resource_edit_table` command
