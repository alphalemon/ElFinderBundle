# ElFinderBundle
The ElFinderBundle encapsulates the elFinder file manager to be used in Symfony2. 

## Installation
Clone this bundle in the vendor/bundles/AlphaLemon directory:

    git clone git://github.com/alphalemon/ElFinderBundle.git vendor/bundles/AlphaLemon/ElFinderBundle

## Configure the ElFinderBundle
Open the AppKernel configuration file and add the bundle to the registerBundles() method:

    public function registerBundles()
    {
        $bundles = array(
            ...
            new new AlphaLemon\ElFinderBundle\AlphaLemonElFinderBundle(),
        )
    }

Register the ElFinderBundle namespaces in `app/autoload.php`:

    $loader->registerNamespaces(array(
        ...
        'AlphaLemon'                     => __DIR__.'/../vendor/bundles',
    ));

Import the routing configuration into the routing.yml file:

    _AlphaLemonElFinderBundle:
        resource: "@AlphaLemonElFinderBundle/Resources/config/routing.yml"

Register the bundle into the Assetic bundles in config.yml:

    # Assetic Configuration
    assetic:
        bundles: [ "AlphaLemonElFinderBundle" ]

Initialize submodules grabbing the ElFinder vendor library. Move inside the ElFinder folder than give this commands:

    git submodule init
    git submodule update

        
To complete the bundle configuration you must install assets as follows:

    app/console assets:install web
    app/console assetic:dump

## Using the object
AlphaLemonElFinderBundle provides a ready to use controller to display the ElFinder:

    http://[yoursite]/al_showElFinder


## Customize elFinder
The default connector has a very minimal configuration, so you would like to configure it on your needs. The elFinder object is loaded into the show.html.twig
template, where all the required assets and initial jquery script are added. To change the configuration you shoud create a new twig template that extends the 
base one:

    /path/to/your/twig/template

    {% extends 'AlphaLemonElFinderBundle:ElFinder:show.html.twig' %}

This template has four blocks you may override:

    {% block stylesheet_files %}{% endblock %}
    
    {% block javascript_files %}{% endblock %}

    {% block init_script %}{% endblock %}

    {% block elfinder_html %}{% endblock %}

The names speak themselves, so if you need to change the init script, you just have to override the init_script block:

    {% block init_script %}
    <script type="text/javascript" charset="utf-8">
        $(document).ready(function() {
            $('<div/>').dialogelfinder({
                    url : '/al_elFinderMediaConnect',
                    lang : 'en',
                    width : 840,
                    destroyOnClose : true
            }).dialogelfinder('instance');
        });
    </script>
    {% endblock %}

Read the elFinder documentation to learn more on the available options

## The connector
The url option declares the connector to use. It is the class where are defined the elFinder options, like the folder where are saved the files and so on. 
In the example above the al_elFinderMediaConnect route is called and the action implementstion might be:

    public function connectMediaAction()
    {
        $connector = $this->container->get('el_finder_media_connector');
        $connector->connect();
    }

You may notice that the connector has been injected into the Dependency Injector Container, and its implementation is:

    <parameters>
        <parameter key="el_finder.media_connector">Path\To\AlphaLemonElFinderMediaConnector</parameter>
    </parameters>

    <services>
        <service id="el_finder_connector" class="%el_finder.media_connector%">
            <argument type="service" id="service_container" />
        </service>
    </services>

The class AlphaLemonElFinderMediaConnector is instantiated into the DIC. Follows a sample of its implementation:

    namespace Path\To\AlphaLemonElFinderMediaConnector;
    
    use AlphaLemon\ElFinderBundle\Core\Connector\AlphaLemonElFinderBaseConnector;

    class AlphaLemonElFinderMediaConnector extends AlphaLemonElFinderBaseConnector
    {
        protected function configure()
        {
            $request = $this->container->get('request');

            $options = array(
                'roots' => array(
                    array(
                        'driver'        => 'LocalFileSystem',   // driver for accessing file system (REQUIRED)
                        'path'          => 'bundles/alphalemonelfinder/files/',         // path to files (REQUIRED)
                        'URL'           => $request->getScheme().'://'.$request->getHttpHost() . '/bundles/alphalemonelfinder/files/', // URL to files (REQUIRED)
                        'accessControl' => 'access'             // disable and hide dot starting files (OPTIONAL)
                    )
                )
            );

            return $options;
        }
    }

The connector extends the AlphaLemonElFinderBaseConnector which requires the derived class to implement a configure() method where the elFinder connector's options
must be declared. This function must return an array of options.