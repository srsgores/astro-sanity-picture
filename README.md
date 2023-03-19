# astro-sanity-picture
An astro component for rendering a responsive <picture> element for an image fetched from [Sanity](https://www.sanity.io). It will generate the element with a set of image sources for optimised resolutions and formats, using sanity's image API to serve the optimised images.

# Usage
---
Minimal example:

```astro
---
import SanityPicture from "astro-sanity-picture";

---
   <SanityPicture
    image={mainBgImage}
    imageUrlBuilder={myImageBuilder}
  /> 
```

Defaults can be set for all picture components

```astro
---
import SanityPicture, {  setSanityPictureDefaults} from "astro-sanity-picture";

setSanityPictureDefaults({ imageUrlBuilder: myImageUrlBuilder })
---
   <SanityPicture
    image={mainBgImage} /> 
```

Attributes of the `<img />` element displayed inside the picture can be set using the `img` property.
For example, to set the sizes attribute, which you'll likely want to set to ensure right-sizing, if your picture is displayed at less than 100vw width:

```astro
---
import SanityPicture, {  setSanityPictureDefaults} from "astro-sanity-picture";

setSanityPictureDefaults({ imageUrlBuilder: myImageUrlBuilder })
---
   <SanityPicture
    image={mainBgImage} 
    img={{sizes: "(min-width:768px) 50vw, 100vw"`}}
    /> 
```

In this example, we are stating that image is to be displayed at half the page width when the page is >= 768px, and at the whole page width otherwise. The browser will then select the source that is appropriate for the image sizing, whether it is 50vw or 100vw.

## Fetching the image from groq
The component will work with images fetched from a simple `groq`  query without fetching any image metadata, eg

```ts
const query = groq`*[_id == 'homePage'][0] {
     ...etc,
     myBackgroundImage,
     ...etc,
  }`
```

However it is able to optimize the generated source sets to be smaller than the original image, and use a low quality placeholder, when the image is fetched with metadata.
To help with this, you can use the `picture` function provided:

```ts
import { picture } from 'astro-sanity-picture/query'

const query = groq`*[_id == 'homePage'][0] {
  ...etc,
  ${picture('myBackgroundImage')},
  ...etc
}`
```

# Component options

- `imageUrlBuilder?: ImageUrlBuilder` - An instance of sanity image url builder to use. If default is set, may be omitted
- `src: SanityImageSource` - The image to display, as a property from a `groq` query
- `sources?: PictureSource[]` - Each `PictureSource` object in the list informs the generation of a `<source />` element for each of the widths generated by the `widths` property. `PictureSource` properties are:
  - `options?: Partial<ImageUrlBuilderOptionsWithAliases>` - Options for the sanity image url builder to apply to this source
  withWebp?: boolean - whether to include a mirrored source in webp format. Default setting is true
  - `...attributes?: Omit<SourceAttributes, "srcset">` - All other attributes that apply to the `<source>` element. Often you will want to set `media` and `sizes`, as in standard usage of the `<picture>` tag.
    - `media?: string` - CSS @media rule that determines when this source applies
    - `sizes?: string;` - comma seperated list of rule - width pairings.
- `widths?: number[] | AutoWidths` - Specifies how to calculate widths for `<source />` elements. You may either specify a list of widths to use, or a an `AutoWidths` type which declares how to automatically determine the widths. 
- `img?: Omit<ImgAttributes, "src">` - Attributes to apply to the base `<img />` element in the picture
- `lqip?: Lqip` - Options for inserting a low quality image placeholder (lqip) as the background image of the element;
  - `enabled: boolean` - Whether to use lqip
  - `transitionDuration: number` - Duration in which to fade in final image once loaded
- `...attributes - PictureAttributes` - Additional attributes to apply to the `<picture />` element;

# Defaults 
- `autowidths`:
```ts
{
  maxWidth: 3840,
  step: 320,
}
```
- `withWebp`: `true`
- `img`: `loading: "lazy"`
- `lqip`: `{ enabled: true, transitionDuration: 350 }`