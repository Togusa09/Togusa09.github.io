

Custom content goes in App_Plugins folder

Template for basic layout

{% highlight html %}
<div class="custom-editor" ng-controller="Section.CustomController as vm">
    <form  val-form-manager>
        <umb-editor-view>
            <umb-editor-header name="vm.pageName"
                               hide-alias="true"
                               hide-description="true"
                               hide-icon="true"
                               name-locked="true">
            </umb-editor-header>

            <umb-editor-container>
            </umb-editor-container>
        </umb-editor-view>
        <umb-editor-footer>
            <umb-editor-footer-content-right>
                <button type="button" class="btn btn-success" ng-disabled="!clinicForm.$valid || vm.loading" ng-click="vm.save(clinicForm)" hotkey="ctrl+s">Save</button>
            </umb-editor-footer-content-right>
        </umb-editor-footer>
    </form>
</div>

{% endhighlight %}