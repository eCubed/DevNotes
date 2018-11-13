# Rxjs - Browser Resize Detector

We have learned quite a few things about `Observable`s and `Subject`s, so now, we're ready to tackle something useful that Rxjs is good for besides trivial examples and
making http calls. We will work on a service that continually detects the browser's pixel width (as we'll continually resize it), but only broadcast to subscribers when the
current size has just crosses a threshhold between small and medium, and medium and large. We'll have 3 distinct sizes for simplicity. So, no, we will not broadcast a size
for every detection of a width change.

## Starting the Service

We've already made the decision that there will be 3 sizes, small, medium, and large. We'll also decide that small is anything upto 400px, medium is 401px to 800px, and
801px and greater will be considered large. The service will return "small", "medium", and "large" when it deems appropriate.

```javascript
import { Injectable } from '@angular/core';

import { Observable, Subject } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ViewportWidthService {

  private currentPixelWidth: number;
  private currentSize: string;

  private sizeChanged$: Subject<string> = new Subject<string>();

  constructor() { }

  private determineWidth() : string {
    if (this.currentPixelWidth <= 400)
      return 'small';
    else if (this.currentPixelWidth >= 401 && this.currentPixelWidth <= 800)
      return 'medium';

    return 'large';
  }

  public size$() : Observable<string> {

    return this.sizeChanged$;
  }
}
```

We need to keep track of the current pixel width and the current size so as the browser width changes, we can update both. We have also provided the logic of how to determine
the size from the current pixel width value. We also set up a `Subject<string>` called $sizeChanged for subscription of when the size changes, and the wrapper function
`size$()` that returns an `Observable<string>` to hide the fact that `sizeChanged$` really is a `Subject<string>`. We do this so that clients cannot explicitly change
the size because that's only this service's job.

## Listening to Width Change