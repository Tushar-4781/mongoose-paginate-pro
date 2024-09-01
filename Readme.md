# **Mongoose Advanced Query Plugin**
#### A Mongoose plugin that provides advanced query functionalities such as filtering, pagination, sorting, population, and geo-spatial searches. This plugin is designed to simplify complex queries in Mongoose by providing a set of flexible options.

### To install the package, run **`npm install mongoose-paginate-pro`**.

# **Features**
**Advanced Filtering**: Pre-aggregation and post-population filtering capabilities.  
**Pagination**: Easily paginate your results with page and limit options.  
**Sorting**: Sort results by specific fields in ascending or descending order.  
**Population**: Populate fields from other collections with custom field selection.  
**Geo-Spatial Searches**: Perform geo-based searches using latitude, longitude, and radius.  
**Custom Pipeline**: Provide your own MongoDB aggregation pipeline to be executed after predefined stages.  

# **Plugin Options**
The plugin accepts three arguments:

# 1. **Filters**
Represents the filters that need to be applied first in the aggregation queries. Use standard MongoDB query syntax for defining these filters.

# 2. **Options**
This object can contain the following keys:

**page**: (Number) The page number you want to retrieve. Default is 1.  
**limit**: (Number) Number of documents per page. Default is 10.  
**sortBy**: (String) Field based on which you want to sort the results.  
**sortOrder**: (String) Order of sorting, either asc for ascending or desc for descending. Default is asc.  
**populate**: (Array of Strings) An array of strings where each string is a pattern that represents the fields to populate and the fields to select from another collection.  
**postPopulateFilters**: (Object) Filters that will be applied once the data is populated. Use standard MongoDB query syntax for these filters.  
**project**: (Object) Define which fields you want to include or exclude in the result using the Mongoose project syntax.  
**pipeline**: (Array) Custom aggregation pipeline stages to be executed after the predefined stages.  

# 3. GeoFilters
Specifically for performing geoNear searches. It accepts an object with the following keys:

**latitude**: (Number) Latitude for the geo search.  
**longitude**: (Number) Longitude for the geo search.  
**fieldName**: (String) The field in the document that stores the geo-coordinates.  
**radius**: (Number) Radius of the search area in miles.  

# Example Usuage 

#### Consider a use case of storing user location
```

const userSchema = new mongoose.Schema(
    {
        name: {
            type: string,
            required: true
        },
        email: {
            type: string,
            required: true
        },
        profilePic: {
            type: string,
            default: null
        },
        role: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'Role',
            required: true,
        }
    }
)

const userLocationSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
      unique: true,
    },
    locationInWords: {
      type: String,
      required: true,
    },
    location: {
      type: {
        type: String,
        enum: ['Point'],
        required: true,
      },
      coordinates: {
        type: [Number],
        required: true,
      },
    },
  },
  {timestamps: true}
);


userLocationSchema.plugin(paginate);
const User = mongoose.model("UserLocation", userLocationSchema);

```
Define a utility function that takes the queryParams and convert them into filters and options
```
const getPaginateConfig = (queryParams = {}) => {
  const { pipeline = [], postPopulateFilters = {}, project = {} ,page = 1, limit = 10, sortBy = 'createdAt', sortOrder = 'desc', ...filters} = queryParams;
  const options = {page, limit, sortBy, sortOrder,pipeline, project, postPopulateFilters};
  return {filters, options};
};
```

```
const { filters, options } = getPaginateConfig();

/* To apply location filter */
const geoFilters = {
    longitude: 42.793549,
    latitude: -101.969624,
    fieldName: location,
    radius: 2
}

/* To populate fields from user */
options.populate = ["user::name,email,profilePic"];

/* To Apply filters after population */
options.postPopulateFilters = { name: { $regex: "mi", $options: 'i' } } ;

/* fiels to exclude */
options.project = { location: 0 };

const data = await UserLocation.paginate(filters,options,geoFilters);
```

# Output 
```
{
    page: 1,
    limit: 10,
    results: [...],
    totalPages: 3,
    totalResults: 26
},
```

## Different scenarios of using populate

```
const groupSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    }, 
    admin: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    inviteCode: {
      type: String,
      required: true,
    },
    members: {
      type: [
        {
          user: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'User',
            required: true,
          },
          role: {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'MemberRole',
            required: true,
          },
        },
      ],
      default: [],
    },
  },
  {timestamps: true}
);

groupSchema.plugin(paginate);

const Group = mongoose.model("Group", groupSchema);

```
### use (.) when population has to be done at nested level  
### use (*) to select all fields from referred collection  

```
const { filters, options } = getPaginateConfig();

options.populate = ["admin::name,email", "members.user::name,email", "members.role::*" ]

const data = await Group.paginate({}, options)
```