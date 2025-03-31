---
author: alc
category:
  - sitecore
date: "2015-12-09T21:18:15+00:00"
draft: "true"
guid: http://laubplusco.net/?p=1627
title: Internal Structure of a Module in Sitecore Modular Architecture
url: /

---
### Internal Module Structure

What does the inside of a Module look like then?

Well this depend on the responsibility of the module, the trends at the time the module was implemented and also the mood and conviction of the developer at the time where the module was first created.

EXAMPLE FROM HABITAT EACH LAYER

EXAMPLE OLD WEBFORMS 3-tiers internal, example DDD internal

_**Recommendation**_, monkey see, monkey do. Developer sees methodology and module structure used in solution, developer follow. This makes the solution easier to maintain. Doing something mediocre all the way is better than doing something in different ways even though your conviction might tell you otherwise.

Is it important to keep the same structure in same layer modules in terms of Modular Architecture?

Not really, Sitecore Habitat presents one way of structuring modules on each layer but even here the structure differ a bit between modules. It is a recommendation based on experience but not part of the architecture. Keeping the same structure in modules within a solution and following the same methodology eases maintenance significantly and should be something you strive for as a solution architect.

At Pentia we sometimes inherit solutions from other partners which we then slowly alter to follow Modular Architecture. This is typically something that happens gradually and not something that is done in one big go. We then blackbox the legacy code in modules that can be replaced. In these cases, the monkey see, monkey do mantra is not something that can be followed.
