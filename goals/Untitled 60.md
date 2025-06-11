void _showReceiverProfileCard() async {

    final stored = await storage.read(key: "receiver_profile_${widget.chatId}");

    final Map<String, dynamic>? profileJson =

        (stored != null) ? jsonDecode(stored) : _receiverProfile;

  

    if (profileJson == null) {

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text("No receiver profile found.")),

      );

      return;

    }

  

    // Convert raw JSON → ProfileData

    profile.ProfileData? profileData;

    try {

      profileData = profile.ProfileData.fromJson(profileJson);

    } catch (e) {

      debugPrint('Error parsing profile JSON: $e');

      ScaffoldMessenger.of(context).showSnackBar(

        const SnackBar(content: Text("Invalid profile data.")),

      );

      return;

    }

  

    // Show modal bottom sheet from 80% to 95%.

    showModalBottomSheet(

      context: context,

      isScrollControlled: true,

      backgroundColor: Colors.transparent,

      barrierColor: Colors.black.withOpacity(0.3),

      builder: (BuildContext bottomCtx) {

        return GestureDetector(

          behavior: HitTestBehavior.opaque,

          onTap: () => Navigator.of(bottomCtx).pop(),

          child: Stack(

            children: [

              // tapping outside closes sheet

              Positioned.fill(child: Container(color: Colors.transparent)),

              Align(

                alignment: Alignment.bottomCenter,

                child: DraggableScrollableSheet(

                  initialChildSize: 0.8, // starts at 80% height

                  minChildSize: 0.8, // cannot go below 80%

                  maxChildSize: 0.95, // can drag up to 95%

  

                  // If on Flutter >= 3.7, you can snap to [0.8, 0.95].

                  snap: true,

                  snapSizes: const [0.8, 0.95],

                  expand: false,

                  builder: (ctx, scrollController) {

                    return ProfileCard(

                      data: profileData!,

                      scrollController: scrollController,

                    );

                  },

                ),

              ),

            ],

          ),

        );

      },

    );

  }