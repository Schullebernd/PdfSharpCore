# Schullebernd/PdfSharpCore
**Schullebernd/PdfSharpCore** is a clone from ststeiger/PdfSharpCore.
The base library can be used in a Xamarin.Forms project to generate PDF files offline. I only made some minor chages to implement a word space handling.

#### This library adds the following features:
- Linespace between words in Drawing.Layout.XTextFormatter can be set manually or in the constructor.

## Not my work
I explicitly want to say that this library is not my work. I only took (forked) it because I needed some minor changes to use it out of the box in a Xamarin app. I couldn't find a pdf library in NuGet that worked out of the box except the one from ststeiger.
The library also contains the MigraDoc port. It's working from my point of view and can also be used. I changed the project output options so that you can easyly build your own nuget packages and use them as offline packages.

## How to use it?
1. Download or clone the project to your local developer machine.
2. Open the project (*.sln) and build the project called *SBMigraDocCore.Rendering*. This builds all libraries and creates NuGet-Packages for all of them.
3. Create a new folder for local nuget packages on your developer machine (e.g. "./LocalNugetPackages").
4. Copy all *.nupgk files from the output folders (./bin/Release/Schullebernd.[...].nupkg) into this local directory.
5. In the NuGet-Manager-Options add this folder as a package source (This has to be done in the solution project thats wants to use the MigraDoc and PdfSharp functions).
6. Now open the Nuget-Manager and switch the source to the new local folder (on the top right corner).
7. Now you should see the list of the self-created nuget packages and can install it.

   7.1. If you only want to use PdfSharpCore, then install Schullebernd.PdfSharpCore package.

   7.2. If you want to use MigraDoc to create pdf files, then install Schullebernd.PdfSharpCore.MigraDocCore package.

## Usage in a Xamarin App
To add pdf generating functions to your Xamarin app you have to take care of some things.
1. You should embedd the fonts you want to use in the pdf document into the shared project of your Xamarin app.
- Add a new folder called ./Fonts to your shared project and copy some fonts into it (I downloaded some fonts from [Google Fonts](https://fonts.google.com/) ).
- At least there should be 4 variants available for each font (e.g. Lato-Bold.ttf, Lato-BoldItalic.ttf, Lato-Italc.ttf and Lato-Regular.ttf).
- Set the build option for this font files to 'Embedded Ressource'
2. Create your own FontResolver in the share project that loads the fonts from the local ressources.
```cs
public class FontResolver : IFontResolver
{
	public string DefaultFontName => "Lato";

	public byte[] GetFont(string faceName)
	{
		using (var ms = new MemoryStream())
		{
			var assembly = typeof(FontResolver).GetTypeInfo().Assembly;
			var resources = assembly.GetManifestResourceNames();
			var resourceName = resources.First(x => x == string.Concat("[ProjectName].Fonts.", faceName)); // replace the [ProjectName] with the name of your shared project
			using (var rs = assembly.GetManifestResourceStream(resourceName))
			{
				rs.CopyTo(ms);
				ms.Position = 0;
				return ms.ToArray();
			}
		}
	}

	public FontResolverInfo ResolveTypeface(string familyName, bool isBold, bool isItalic)
	{
		if (familyName.Equals("Lato", StringComparison.CurrentCultureIgnoreCase))
		{
			if (isBold && isItalic)
			{
				return new FontResolverInfo("Lato-BoldItalic.ttf");
			}
			else if (isBold)
			{
				return new FontResolverInfo("Lato-Bold.ttf");
			}
			else if (isItalic)
			{
				return new FontResolverInfo("Lato-Italic.ttf");
			}
			else
			{
				return new FontResolverInfo("Lato-Regular.ttf");
			}
		}
		return null;
	}
}
```
3. Initialize the font resolver only once
```cs
// Init the self made FontResolver (only once)
if (!FontResolverAlreadySet)
{
    GlobalFontSettings.FontResolver = new FontResolver();
    FontResolverAlreadySet = true;
}
```




# Here is the original readme content from ststeiger/PdfSharpCore
# PdfSharpCore

**PdfSharpCore** is a partial port of [PdfSharp.Xamarin](https://github.com/roceh/PdfSharp.Xamarin/) for .NET Standard
Additionally MigraDoc has been ported as well (from version 1.32).
Images have been implemented with [ImageSharp](https://github.com/JimBobSquarePants/ImageSharp/), which is still in Alpha. They State on their readme that it is still in Alpha status and shouldn't be used in productive environments. Since I didn't find any good alternatives it's still used.

ImageSharp beeing Alpha isn't a big issure either since this code isn't by far done yet. So please chime in ;)

**PdfSharp.Xamarin** is a partial port of [PdfSharp](http://www.pdfsharp.net/) for iOS and Android using Xamarin, it allows for creation and modification of PDF files.



###### Example project 

There was an example project here. <br />
I've removed it from this project, and put it into a separate solution. 
You can find it [here](https://github.com/ststeiger/Stammbaum).<br />
There's a default font-resolver in [FontResolver.cs](https://github.com/ststeiger/PdfSharpCore/blob/master/PdfSharpCore/Utils/FontResolver.cs).<br />
It should work on Windows, Linux, OSX and Azure. <br />
Some limitations apply. <br />
See open issues. 


## Example usage 

```cs
//See the "Example" Project for a MigraDoc example
static void Main(string[] args)
{
    GlobalFontSettings.FontResolver = new FontResolver();
    
    var document = new PdfDocument();
    var page = document.AddPage();
    var gfx = XGraphics.FromPdfPage(page);
    var font = new XFont("OpenSans", 20, XFontStyle.Bold);
            
    gfx.DrawString("Hello World!", font, XBrushes.Black, new XRect(20, 20, page.Width, page.Height), XStringFormats.Center);

    document.Save("test.pdf");
}

//This implementation is obviously not very good --> Though it should be enough for everyone to implement their own.
public class FontResolver : IFontResolver
{
    public byte[] GetFont(string faceName)
    {
        using(var ms = new MemoryStream())
        {
            using(var fs = File.Open(faceName, FileMode.Open))
            {
                fs.CopyTo(ms);
                ms.Position = 0;
                return ms.ToArray();
                }
            }
        }
        public FontResolverInfo ResolveTypeface(string familyName, bool isBold, bool isItalic)
        {
            if (familyName.Equals("OpenSans", StringComparison.CurrentCultureIgnoreCase))
            {
                if (isBold && isItalic)
                {
                    return new FontResolverInfo("OpenSans-BoldItalic.ttf");
                }
                else if (isBold)
                {
                    return new FontResolverInfo("OpenSans-Bold.ttf");
                }
                else if (isItalic)
                {
                    return new FontResolverInfo("OpenSans-Italic.ttf");
                }
                else
                {
                    return new FontResolverInfo("OpenSans-Regular.ttf");
                }
            }
            return null;
        }
    }
}
```

## License

Copyright (c) 2005-2007 empira Software GmbH, Cologne (Germany)  
Modified work Copyright (c) 2016 David Dunscombe

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
