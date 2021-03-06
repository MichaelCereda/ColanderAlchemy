.. _examples:

Examples
========

Suppose you have these SQLAlchemy mapped classes::

    from sqlalchemy import Column
    from sqlalchemy import Enum
    from sqlalchemy import ForeignKey
    from sqlalchemy import Integer
    from sqlalchemy import Unicode
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import relationship


    Base = declarative_base()


    class Person(Base):
        __tablename__ = 'persons'

        id = Column(Integer, primary_key=True)
        name = Column(Unicode(128), nullable=False)
        surname = Column(Unicode(128), nullable=False)
        gender = Column(Enum(u'M', u'F'))
        age = Column(Integer)
        phones = relationship('Phone')
        friends = relationship('Friend')


    class Phone(Base):
        __tablename__ = 'phones'

        person_id = Column(Integer, ForeignKey('persons.id'), primary_key=True)
        number = Column(Unicode(128), primary_key=True)
        location = Column(Enum(u'home', u'work'))


    class Friend(Base):
        __tablename__ = 'friends'

        person_id = Column(Integer, ForeignKey('persons.id'), primary_key=True)
        friend_of = Column(Integer, ForeignKey('persons.id'), primary_key=True)
        rank = Column(Integer, default=0)


The code you need to create the Colander schema for ``Person`` is::

    import colander


    class Friend(colander.MappingSchema):
        person_id = colander.SchemaNode(colander.Int())
        friend_of = colander.SchemaNode(colander.Int())
        rank = colander.SchemaNode(colander.Int(), missing=0, default=0)


    class Phone(colander.MappingSchema):
        person_id = colander.SchemaNode(colander.Int())
        number = colander.SchemaNode(colander.String(),
                                     colander.Length(0, 128))
        location = colander.SchemaNode(colander.String(),
                                       validator=colander.OneOf(['home', 'work']),
                                       missing=None)


    class Friends(colander.SequenceSchema):
        friend = Friend()


    class Phones(colander.SequenceSchema):
        phone = Phone()


    class Person(colander.MappingSchema):
        id = colander.SchemaNode(colander.Int())
        name = colander.SchemaNode(colander.String(),
                                   colander.Length(0, 128))
        surname = colander.SchemaNode(colander.String(),
                                      colander.Length(0, 128))
        age = colander.SchemaNode(colander.Int(), missing=None)
        gender = colander.SchemaNode(colander.String(),
                                     validator=colander.OneOf(['M', 'F']),
                                     missing=None)
        friends = Friends(missing=[], default=[])
        phones = Phones(missing=[], default=[])


        person = Person()


The code you need to get the same Colander schema for ``Person`` using ColanderAlchemy is::

    from colanderalchemy import SQLAlchemyMapping


    person = SQLAlchemyMapping(Person)