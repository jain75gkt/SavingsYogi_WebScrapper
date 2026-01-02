try {
  (function (parent, name, context, definition) {
    if (!context || !context[parent] || !Array.prototype.filter) return;
    context[parent]['plugins']['glance'] = definition();
  })('bactm', 'glance', typeof window !== 'undefined' ? window : null, function () {

    var parent = 'bactm';
    var ba = window[parent];
    var win = window;
    var doc = document || {};
    var version = '6.48.0';
    var ddo = win.digitalData;
    var LOG_LEVEL = {
      OFF: 10,
      FATAL: 5,
      ERROR: 4,
      WARN: 3,
      INFO: 2,
      DEBUG: 1
    };

    if (!ddo) return;

    /**
     * @method masking
     * @public
     */
    var masking = function (maskSelectors = window.digitalData.page.attributes.glance.maskSelectors) {
      try {
        const allElementsToMask = []
        maskSelectors.forEach(selector => {
          const allOfThisSelector = document.querySelectorAll(selector);
          allOfThisSelector.forEach(item => allElementsToMask.push(item));
        });

        allElementsToMask.forEach(element => {
          element.classList.add("glance_masked")
        })
      } catch (err) {
        console.log("Error: glance_masked");
        bactm.reportError(err);
      }
    };

    /**
     * @method addGlanceLoadedClass
     * @private
     */
    var addGlanceLoadedClass = function () {
      try {
        const $glanceButtons = document.querySelectorAll('.glance-button');
        for (var i = 0; i < $glanceButtons.length; i++) {
          $glanceButtons[i].classList.add('glance_available');
        }
      } catch (err) {
        console.log("Error: glance available");
        bactm.reportError(err);
      }
    };

    /**
     * @method addGlanceLoadedClass
     * @private
     */
    var addGlanceNotLoadedClass = function () {
      try {
        console.log('bactm.plugin.glance: Glance Not Loaded classes');
        const $glanceButtons = document.querySelectorAll('.glance-button');
        for (var i = 0; i < $glanceButtons.length; i++) {
          $glanceButtons[i].classList.add('glance_unavailable');
        }
      } catch (err) {
        console.log("Error: glance_unavailable");
        bactm.reportError(err);
      }

    };

    /**
     * @method auth
     * @param params {object}
     * @public
     */
    var authSetup = function (params) {
      console.log('bactm.plugin.glance: Auth Setup called', params);
      const presencevisitor = new GLANCE.Presence.Visitor({
        visitorid: params || digitalData.user.PartyID
      });
      presencevisitor.presence();
      presencevisitor.connect();
    };

    /**
     * @method unauth
     * @param params {object}
     * @public
     */
    var startServer = function (params) {
      console.log('bactm.plugin.glance: Start Server called', params);
    };

    /**
     * @method addButtonListeners
     * @public
     */
    const addButtonListeners = () => {
      try {
        var glanceLinks = document.getElementsByClassName('browse-with-specialist');
        var cobrowseLink = document.getElementById('cobrowse');
        var glanceTrigger = () => GLANCE.Cobrowse.Visitor.showTerms({ video: 'off' });
        if (glanceLinks.length !== 0) { // Check for class first
          for (let j = 0; j < glanceLinks.length; j++) {
            glanceLinks[j].addEventListener('click', glanceTrigger);
          }
        } else if (cobrowseLink) { // Fallback to ID (legacy)
          cobrowseLink.addEventListener('click', glanceTrigger)
        } else {
          console.log('Glance Button not available.')
        }
      } catch (err) {
        console.log('Error: Glance Trigger')
        bactm.reportError(err)
      }
    }

    /**
     * @method appendGlanceScriptAndPrepCallback
     * @public
     */
    const appendGlanceScriptAndPrepCallback = (callback) => {
      const {ws, groupID, src, env} = digitalData?.page?.attributes?.glance;
      var glance = document.createElement('script');
      var head = document.getElementsByTagName('head')[0];
      glance.id = 'glance-cobrowse';
      glance.type = 'text/javascript';
      glance.src = src;
      glance.setAttribute('data-site', env)
      glance.setAttribute('data-presence', 'api')
      // set groupID for prod/staging
      glance.setAttribute('data-groupid', groupID)
      // set data-ws for prod/staging
      glance.setAttribute('data-ws', ws)

      if (document.location.host.includes('.ml.com')) {
        glance.setAttribute('data-cookiedomain', 'ml.com')
      }

      // once glance script is fully initiated, run our callback functions
      glance.onload = () => {
        // we need to wait for the subsequent script loaded from GlanceCobrowseLoader
        bactm.scriptReady('GlancePresenceVisitor', { childList: true }, callback)
      };
      head.appendChild(glance)
    }

    // you can run this code block to set dummy cookies for testing.
    // Presence:
    // const bananas = new bactm.Cookies();
    // bananas.set('cb_session', 'bananas');
    // bananas.set('cb_type', 'presence');
    // Code:
    // const bananas = new bactm.Cookies();
    // bananas.set('cb_type', 'code');

    /**
     * @method init
     * @public
     */
    var _init = function () {
      ba._log('glance plugin v' + version + ' initializing.', LOG_LEVEL.INFO);
      addButtonListeners();


      const cookies = new ba.Cookies();
      const cbTypeCookie = cookies.get('cb_type')
      const cbSessionCookie = cookies.get('cb_session');
      const ddoPartyID = window?.digitalData?.user?.PartyID;

      const presenceScript = (cb) => {
        if (!cb) return;
        const scripts = document.getElementsByTagName('script');
        const scriptsArray = Array.from(scripts);
        const presenceScript = scriptsArray.filter(script => script.src.includes('GlancePresenceVisitor'));
        if (presenceScript.length > 0) return presenceScript[0].onload = () => { cb(); };
        return;
      }

      let mobilePartyID = (cbTypeCookie === 'presence' && cbSessionCookie.length > 0) ? cbSessionCookie : false;

      const { isWealth } = window.digitalData.page.attributes.glance;

      // Trigger cross-domain functionality for wealth users on desktop
      if (!mobilePartyID && isWealth) {
        GLANCE.Cobrowse.Visitor.addEventListener('sessionstart', () => GLANCE.Cobrowse.VisitorUI.promptCrossDomain());
      }

      // prefer to use MDA-passed value from cookie. Else use DDO value, else return false.
      const evaluatePartyID = () => (mobilePartyID ? mobilePartyID : (ddoPartyID ? ddoPartyID : false));
      // trigger masking. Most masking is done via the salesforce glance dashboard, but this has always been around, so not removing it.
      masking();
      // if Glance defined, allow button to show. This is just on one page owned by Global Tenants team.
      if (!window.GLANCE) return addGlanceNotLoadedClass();
      addGlanceLoadedClass();
      // mobile partyID passed from MDA, use that, else use DDO partyID, else return false;
      var glancePartyID = evaluatePartyID();
      // proactively show code. This is mobile use-case only.
      if (cbTypeCookie === 'code' && GLANCE?.Cobrowse?.Visitor?.startSession) {
        // show code
        GLANCE.Cobrowse.Visitor.startSession();
        // delete cookie so we don't show it again on subsequent page loads.
        cookies.remove('cb_type');
      }
      // if we have a partyID, wait until presence script fully loaded, then publish presence.
      if (glancePartyID) return presenceScript(()=> {authSetup(glancePartyID)})
      return
    };

    /**
     * Run init
     */
    try {
      appendGlanceScriptAndPrepCallback(_init);
      bactm.on('vendor:glance:auth', authSetup)
    } catch (e) {
      bactm.reportError(e)
    }

    /**
     * Public methods
     */
    return {
      startServer,
      addButtonListeners,
      addGlanceLoadedClass
    };

  });
} catch (e) {
  bactm.reportError(e)
}
