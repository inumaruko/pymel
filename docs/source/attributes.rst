
.. currentmodule:: pymel

Attributes
==========

The `Attribute` class is your one-stop shop for all attribute related functions. Those of us who have spent time using MEL
have become familiar with all the many commands for operating on attributes.  This class gathers them all into one
place. If you forget or are unsure of the right method name, just ask for help by typing `help(Attribute)`.  

For the most part, the names of the class equivalents to the maya.cmds functions follow a fairly simple pattern:
`setAttr` becomes `Attribute.set`, `getAttr` becomes `Attribute.get`, `connectAttr` becomes `Attribute.connect` and so on.  
Here's a simple example showing how the Attribute class is used in context.

    >>> from pymel import *
    >>> cam = general.PyNode('persp')
    >>> if cam.visibility.isKeyable() and not cam.visibility.isLocked():
    ...     cam.visibility.set( True )
    ...     cam.visibility.lock()
    ... 
    >>> print cam.v.type()      # shortnames also work    
    bool

Accessing Attributes
--------------------

You can access an attribute class in three ways.  The first two require that you already have a `PyNode` object.

Shorthand
~~~~~~~~~

The shorthand method is the most visually appealing and readable -- you simply access the maya attribute as a normal python attribute --
but it has one major drawback: **if the attribute that you wish to acess has the same name as one of the attributes or methods of the 
python class then it will fail**. 

    >>> cam  # continue from where we left off above
    Transform(u'persp')
    >>> cam.visibility # long name access
    Attribute(u'persp.visibility')
    >>> cam.v # short name access
    Attribute(u'persp.visibility')
    
Keep in mind, that regardless of whether you use the long or short name of the attribute, you are accessing the same underlying API object.
If you need the attribute formatted as a string in a particular way, use `Attribute.name`, `Attribute.longName`, `Attribute.shortName`,
`Attribute.plugAttr`, or `Attribute.lastPlugAttr`.


attr Method
~~~~~~~~~~~
The attr method is the safest way to access an attribute, and can even be used to access attributes that conflict with 
python methods, which would fail using shorthand syntax. This method is passed a string which
is the name of the attribute to be accessed. 
    
    >>> cam.attr('visibility')
    Attribute(u'persp.visibility')

Unlike the shorthand syntax, this method is capable of being passed attributes which are passed in as variables:        
    
    >>> for axis in ['scaleX', 'scaleY', 'scaleZ']: 
    ...     cam.attr( axis ).lock()          

Direct Instantiation
~~~~~~~~~~~~~~~~~~~~
The last way of getting an attribute is by directly instantiating the class. You can pass the attribute name as a string, or if you have one handy,
pass in an api MPlug object.  If you don't know whether the string name represents a node or an attribute, you can always instantiate via the `PyNode`
class, which will determine the appropriate class automaticallly.

explicitly request an Attribute:

    >>> Attribute( 'persp.visibility' ) 
    Attribute(u'persp.visibility')
    
let general.PyNode figure it out for you:

    >>> general.PyNode( 'persp.translate' ) 
    Attribute(u'persp.translate')


Setting Attributes Values
-------------------------

To set the value of an attribute, you use the `Attribute.set` method.

    >>> cam.translateX.set(0)
    
to set an attribute that expects a double3, you can use any iterable with 3 elements:

    >>> cam.translate.set([4,5,6])
    >>> cam.translate.set(datatypes.Vector([4,5,6]))

Getting Attribute Values
------------------------
To get the value of an attribute, you use the `Attribute.get` method. Keep in mind that, where applicable, the values returned will 
be cast to pymel classes. This example shows that rotation (along with translation and scale) will be returned as a `Vector <pymel.core.datatypes.Vector>`.

    >>> t = cam.translate.get()
    >>> print t
    [4.0, 5.0, 6.0]
    >>> # translation is returned as a vector class
    >>> print type(t) 
    <class 'pymel.core.datatypes.Vector'>
    
`Attribute.set` is flexible in the types that it will accept, but `Attribute.get` will always return the same type 
for a given attribute. This can be a potential source of confusion:
    
    >>> value = [4,5,6]
    >>> cam.translate.set(value)
    >>> result = cam.translate.get()
    >>> value == result
    False
    >>> # why is this? because result is a Vector and value is a list
    >>> # use `Vector.isEquivalent` or cast the list to a `Vector`
    >>> result == datatypes.Vector(value)
    True
    >>> result.isEquivalent(value)
    True

Connecting Attributes
---------------------
As you might expect, connecting and disconnecting attributes is pretty straightforward.
            
    >>> cam.rotateX.connect( cam.rotateY )
    >>> cam.rotateX.disconnect( cam.rotateY )

there are also handy operators for `connection <Attribute.__rshift__>` and `disconnection <Attribute.__floordiv__>`

    >>> c = polyCube(name='testCube')[0]        
    >>> cam.tx >> c.tx    # connect
    >>> cam.tx.outputs()
    [Transform(u'testCube')]
    >>> cam.tx // c.tx    # disconnect
    >>> cam.tx.outputs()
    []
        