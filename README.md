# Chomp

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Emoji_u1f40a.svg/600px-Emoji_u1f40a.svg.png" width="200px" align="right" />

### Take the bite out of React and Redux

React is an unopinionated framework. This means that when it comes to building apps, a lot of time is spent deciding on the best way to approach developing specific features e.g. should component files be named `HotdogList.js`, `hotdogList.js`, `hotdog.list.js` etc. This results in hours of frustration, sloppy code, and a whole bunch of wasted time and *many* sleepless nights.

Ruby on Rails adopted the doctrine of **convention over configuration**. This basically means that they have strict guidelines on how features may be developed. They show exactly how variables need to be defined, how validations should be made, what the folder structure must be, and everything inbetween. As such, because developers don't need to think about these trivial development decisions, they have been able to:

1. Speed up development time
2. Reduced barriers to entry
3. Free up more time up for playing RuneScape

I'm not saying we should adopt the same conventions as Rails but we should definitely adopt the doctrine. Therefore, without futher adue, here is Chomp; the convention over configuration guidelines for React and Redux applications.

## File Structure

Organise your files by feature. This makes it a lot easier to:

1. Find the files you need
2. Name your files
3. Manage folder sizes on large scale applications
4. Share specific modules between multiple applications

The folder structure rules are as such:

1. Root folders should be the singular name of the feature
2. All files that do not fit in a specific feature folder go in the `shared` folder
3. Helper files should be formatted as `<feature>.<type>.js` e.g. `hotdog.reducer.js`
4. There should be a `components` and `containers` set of folders in each feature
5. Container (smart) file names should be formatted as `<Feature><Action>.js`
6. Component (dumb) file names should be formatted as `<Description><Type>.js`

```
src
+-- hotdog
    +-- components // do not include feature name in component name as should not use outside feature
        +-- GoodButton.js
    +-- containers // format of file names should be <Feature><Action>
        +-- HotdogList.js
        +-- HotdogUpdate.js
    +-- hotdog.reducer.js
    +-- hotdog.service.js
+-- drink
    +-- components
        +-- ListItem.js
        +-- CupWrap.js
        +-- EditForm.js
    +-- containers
        +-- DrinkList.js
        +-- DrinkCreate.js
        +-- DrinkUpdate.js
+-- shared // where all files that don't belong to a specific feature should go
    +-- components
        +-- CommonButton.js
    +-- containers
        +-- App.js
    +-- utils.helper.js
```

## Reducers

## Services

## Containers vs Components

## Routing

## Forms
